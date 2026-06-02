# Advanced Collision Detection - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/advanced_collision_detection

## Pipeline Overview

1. **BroadPhase** - detects potentially intersecting collider pairs
2. **NarrowPhase** - computes contact points and generates events
3. **Constraints Solver** - computes contact forces from contact points

Two graph structures store results:
- **Contact graph** - non-sensor pairs + contact points
- **Intersection graph** - sensor pairs + boolean intersection results

## Collision and Contact Force Events

```rust
fn display_events(
    mut collision_events: EventReader<CollisionEvent>,
    mut contact_force_events: EventReader<ContactForceEvent>,
) {
    for event in collision_events.read() {
        println!("Collision: {:?}", event);
    }
    for event in contact_force_events.read() {
        println!("Contact force: {:?}", event);
    }
}
```

**Requirements:**
- Collision events: at least one collider must have `ActiveEvents::COLLISION_EVENTS`
- Contact force events: at least one collider must have `ActiveEvents::CONTACT_FORCE_EVENTS` + `ContactForceEventThreshold`

Event flags:
- `CollisionEventFlags::SENSOR` - at least one collider is a sensor
- `CollisionEventFlags::REMOVED` - collision stopped due to collider removal

## Contact Graph

```rust
fn display_contact_info(rapier_context: ReadRapierContext) {
    let ctx = rapier_context.single().unwrap();

    // Contact pair between two specific colliders
    if let Some(contact_pair) = ctx.contact_pair(entity1, entity2) {
        if contact_pair.has_any_active_contact() {
            // Has solver contacts with computed forces
        }

        for manifold in contact_pair.manifolds() {
            println!("Normal: {}", manifold.normal());
            for contact in manifold.points() {
                println!("Local point 1: {:?}", contact.local_p1());
                println!("Distance: {:?}", contact.dist());
                println!("Impulse: {}", contact.raw.data.impulse);
                println!("Friction impulse: {}", contact.raw.data.tangent_impulse);
            }
            for solver_contact in &manifold.raw.data.solver_contacts {
                println!("Solver point: {:?}", solver_contact.point);
                println!("Solver distance: {:?}", solver_contact.dist);
            }
        }
    }

    // All contact pairs for one collider
    for contact_pair in ctx.contact_pairs_with(entity) {
        // Process each pair
    }
}
```

**Important:**
- Fields ending in `1` relate to `contact_pair.collider1()`, fields ending in `2` to `contact_pair.collider2()`
- The pair may have entity handles swapped from query order
- Contact pair existing != actually touching (check `has_any_active_contact` or manifold points)
- Geometric contacts are in local space; solver contacts are in world space
- Non-convex shapes (trimesh, compound) produce multiple manifolds

## Intersection Graph

```rust
// Specific pair (at least one must be a sensor)
if ctx.intersection_pair(entity1, entity2) == Some(true) {
    println!("Intersecting!");
}

// All intersection pairs for one collider
for (c1, c2, intersecting) in ctx.intersection_pairs_with(entity) {
    if intersecting {
        println!("{:?} and {:?} intersect", c1, c2);
    }
}
```

## Physics Hooks

Custom callbacks for filtering contacts and modifying contacts.

### Setup

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(RapierPhysicsPlugin::<MyPhysicsHooks>::default())
        .run();
}

#[derive(SystemParam)]
struct MyPhysicsHooks;

impl BevyPhysicsHooks for MyPhysicsHooks {
    fn filter_contact_pair(&self, context: PairFilterContextView) -> Option<SolverFlags> {
        // Return None to skip, Some(SolverFlags) to compute
    }

    fn filter_intersection_pair(&self, context: PairFilterContextView) -> bool {
        // Return true to compute intersection
    }

    fn modify_solver_contacts(&self, context: ContactModificationContextView) {
        // Modify contact properties
    }
}
```

Use `NoUserData` when no hooks needed: `RapierPhysicsPlugin::<NoUserData>::default()`.

### Contact and Intersection Filtering

Requires `ActiveHooks::FILTER_CONTACT_PAIRS` or `ActiveHooks::FILTER_INTERSECTION_PAIR` on at least one collider.

```rust
#[derive(SystemParam)]
struct SameTagFilter<'w, 's> {
    tags: Query<'w, 's, &'static CustomFilterTag>,
}

impl BevyPhysicsHooks for SameTagFilter<'_, '_> {
    fn filter_contact_pair(&self, ctx: PairFilterContextView) -> Option<SolverFlags> {
        if self.tags.get(ctx.collider1()).ok().copied()
            == self.tags.get(ctx.collider2()).ok().copied() {
            Some(SolverFlags::COMPUTE_IMPULSES)
        } else {
            None
        }
    }

    fn filter_intersection_pair(&self, ctx: PairFilterContextView) -> bool {
        self.tags.get(ctx.collider1()).ok().copied()
            == self.tags.get(ctx.collider2()).ok().copied()
    }
}
```

### Contact Modification

Requires `ActiveHooks::MODIFY_SOLVER_CONTACTS` on at least one collider.

Use cases: conveyor belts (tangent_velocity), one-way platforms (delete contacts by normal), non-uniform friction/restitution.

```rust
impl BevyPhysicsHooks for MyPhysicsHooks {
    fn modify_solver_contacts(&self, context: ContactModificationContextView) {
        *context.raw.normal = -*context.raw.normal;  // flip normal
        context.raw.solver_contacts.swap_remove(0);   // delete contact

        for sc in &mut *context.raw.solver_contacts {
            sc.friction = 0.3;
            sc.restitution = 0.4;
            sc.tangent_velocity.x = 10.0;
        }

        // Persistent user data across timesteps
        *context.raw.user_data += 1;
    }
}
```

**Warning:** Cannot add new contacts, only modify/delete existing ones.
