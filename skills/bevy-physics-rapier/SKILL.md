---
created: 2026-05-19
modified: 2026-05-28
name: bevy-physics-rapier
description: "Bevy physics with Rapier 0.30: rigid bodies, colliders, joints, character controllers, scene queries, physics hooks, and advanced collision detection. Use when adding physics to Bevy games, working with bevy_rapier2d or bevy_rapier3d, or handling collisions and rigid body dynamics."
---

# Bevy Physics with Rapier (0.30)

Expert knowledge for integrating the Rapier physics engine with Bevy via `bevy_rapier2d` or `bevy_rapier3d` (version 0.30).

## Quick Start

```rust
// Cargo.toml
// bevy_rapier2d = { version = "0.30", features = ["debug-render-2d"] }
// bevy_rapier3d = { version = "0.30", features = ["debug-render-3d"] }

use bevy::prelude::*;
use bevy_rapier3d::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(RapierPhysicsPlugin::<NoUserData>::default())
        .add_plugins(RapierDebugRenderPlugin::default())
        .add_systems(Startup, setup_physics)
        .run();
}

fn setup_physics(mut commands: Commands) {
    // Ground (no RigidBody = Fixed)
    commands
        .spawn(Collider::cuboid(100.0, 0.1, 100.0))
        .insert(Transform::from_xyz(0.0, -2.0, 0.0));

    // Bouncing ball
    commands
        .spawn(RigidBody::Dynamic)
        .insert(Collider::ball(0.5))
        .insert(Restitution::coefficient(0.7))
        .insert(Transform::from_xyz(0.0, 4.0, 0.0));
}
```

## Cargo Features

```toml
bevy_rapier3d = { version = "0.30", features = [
    "simd-stable",           # SIMD (stable Rust, via wide crate)
    "debug-render-3d",       # Debug rendering plugin
    "parallel",              # Parallel physics via rayon
    "serde-serialize",       # Serde support
    "enhanced-determinism",  # Cross-platform determinism
] }
```

## Core Concepts

### Rigid Body Types

| Type | Behavior | Use Case |
|------|----------|----------|
| `RigidBody::Dynamic` | Full physics simulation | Players, projectiles, debris |
| `RigidBody::Fixed` | Immovable, infinite mass | Ground, walls, static obstacles |
| `RigidBody::KinematicPositionBased` | User sets position; velocity deduced | Moving platforms, elevators |
| `RigidBody::KinematicVelocityBased` | User sets velocity; position deduced | Conveyors, scrolling backgrounds |

**Key:** Kinematic bodies ignore ALL contact forces. They pass through walls. Use character controller or scene queries for obstacle avoidance.

### Collider Shapes

```rust
// 2D
Collider::ball(radius)
Collider::cuboid(half_width, half_height)
Collider::capsule(half_height, radius)
Collider::segment(p1, p2)
Collider::convex_hull(points)
Collider::polyline(vertices, indices)
Collider::heightfield(heights, scale)
Collider::compound(vec![(pos, rot, shape), ...])

// 3D
Collider::ball(radius)
Collider::cuboid(half_x, half_y, half_z)
Collider::capsule(half_height, radius)
Collider::cylinder(half_height, radius)
Collider::cone(half_height, radius)
Collider::convex_hull(points)
Collider::convex_mesh(points, indices)
Collider::trimesh(vertices, indices)
Collider::heightfield(heights_matrix, scale)
Collider::voxels_from_points(cell_size, points)
Collider::compound(vec![(pos, rot, shape), ...])
Collider::round_cuboid(half_x, half_y, half_z, border_radius)
```

### Collider Attachment

```rust
// Same entity as RigidBody
commands.spawn(RigidBody::Dynamic).insert(Collider::ball(0.5));

// Multiple colliders as children (relative positioning)
commands
    .spawn((RigidBody::Dynamic, GlobalTransform::default()))
    .with_children(|children| {
        children.spawn(Collider::ball(0.5))
            .insert(Transform::from_xyz(0.0, 0.0, -1.0));
    });

// Standalone (implicit Fixed)
commands.spawn(Collider::cuboid(1.0, 2.0));
```

### Sensors

```rust
commands.spawn(Collider::ball(0.5)).insert(Sensor);
```

Sensors generate intersection events only, no contact forces. Use for triggers, hitboxes, zones.

## Moving Rigid Bodies

### Dynamic Bodies - Use Forces or Velocity

```rust
// Set velocity directly
commands.spawn(RigidBody::Dynamic).insert(Velocity {
    linvel: Vec3::new(0.0, 2.0, 0.0),
    angvel: Vec3::ZERO,
});

// Apply persistent force
commands.spawn(RigidBody::Dynamic).insert(ExternalForce {
    force: Vec3::new(10.0, 0.0, 0.0),
    torque: Vec3::ZERO,
});

// Apply one-time impulse
commands.spawn(RigidBody::Dynamic).insert(ExternalImpulse {
    impulse: Vec3::new(100.0, 0.0, 0.0),
    torque_impulse: Vec3::ZERO,
});
```

### Kinematic Bodies - Set Transform or Velocity

```rust
// KinematicPositionBased: modify Transform
fn move_platform(mut transforms: Query<&mut Transform, With<RigidBody::KinematicPositionBased>>) {
    for mut t in transforms.iter_mut() {
        t.translation.y += 0.01;
    }
}

// KinematicVelocityBased: set Velocity
fn move_platform(mut velocities: Query<&mut Velocity, With<RigidBody::KinematicVelocityBased>>) {
    for mut v in velocities.iter_mut() {
        v.linvel = Vec3::new(0.0, 1.0, 0.0);
    }
}
```

## Character Controller

For player characters that need obstacle avoidance:

```rust
// Setup
commands
    .spawn(RigidBody::KinematicPositionBased)
    .insert(Collider::capsule_y(0.4, 0.3))
    .insert(KinematicCharacterController {
        up: Vec3::Y,
        max_slope_climb_angle: 45_f32.to_radians(),
        min_slope_slide_angle: 30_f32.to_radians(),
        autostep: Some(CharacterAutostep {
            max_height: CharacterLength::Absolute(0.5),
            min_width: CharacterLength::Absolute(0.2),
            include_dynamic_bodies: true,
        }),
        snap_to_ground: Some(CharacterLength::Absolute(0.5)),
        ..default()
    });

// Each frame: set desired translation
controller.translation = Some(Vec3::new(x, 0.0, z) * delta_secs);

// Read results
let output: &KinematicCharacterControllerOutput;
output.effective_translation;  // actual movement
output.grounded;               // is on ground?
output.collisions;             // chronological collision list
```

See `references/character_controller.md` for full details.

## Collision Groups

32 groups (u32 bitmasks). Preferred over solver groups (skips more computation):

```rust
commands
    .spawn(Collider::ball(0.5))
    .insert(CollisionGroups::new(
        Group::GROUP_1 | Group::GROUP_2,  // membership
        Group::GROUP_1,                    // filter (who to interact with)
    ));
```

## Collision Events

```rust
// Enable on collider
commands.spawn(Collider::ball(0.5))
    .insert(ActiveEvents::COLLISION_EVENTS);

// Read events
fn on_collision(mut events: EventReader<CollisionEvent>) {
    for event in events.read() {
        // event.entity1, event.entity2, event.started, event.flags
    }
}
```

## Scene Queries

```rust
fn query_system(rapier_context: ReadRapierContext) {
    let ctx = rapier_context.single().unwrap();

    // Ray cast
    if let Some((entity, toi)) = ctx.cast_ray(origin, dir, max_toi, solid, filter) {
        // Handle hit
    }

    // Shape cast (sweep test)
    if let Some((entity, hit)) = ctx.cast_shape(pos, rot, vel, &shape, options, filter) {
        // Handle hit
    }

    // Point projection
    if let Some((entity, proj)) = ctx.project_point(point, solid, filter) {
        // proj.point, proj.is_inside
    }

    // Intersection test
    ctx.intersections_with_shape(pos, rot, &shape, filter, |entity| { true });
}
```

See `references/scene_queries.md` for QueryFilter options and full API.

## Joints

```rust
// Revolute (hinge)
let joint = RevoluteJointBuilder::new(Vec3::X)
    .local_anchor1(Vec3::ZERO)
    .local_anchor2(Vec3::ZERO);
commands.spawn(RigidBody::Dynamic)
    .insert(ImpulseJoint::new(parent_entity, joint));

// Prismatic (slider) with limits and motor
let joint = PrismaticJointBuilder::new(Vec3::X)
    .limits([-2.0, 5.0])
    .motor_velocity(0.1, 0.05);

// Spherical (ball socket)
let joint = SphericalJointBuilder::new()
    .local_anchor1(Vec3::ZERO);
```

See `references/joints.md` and `references/joint_constraints.md` for MultibodyJointSet vs ImpulseJointSet.

## Physics Hooks

Custom contact filtering and modification:

```rust
#[derive(SystemParam)]
struct MyHooks;

impl BevyPhysicsHooks for MyHooks {
    fn filter_contact_pair(&self, ctx: PairFilterContextView) -> Option<SolverFlags> {
        Some(SolverFlags::COMPUTE_IMPULSES)
    }

    fn modify_solver_contacts(&self, ctx: ContactModificationContextView) {
        // Modify friction, restitution, tangent_velocity, etc.
    }
}

// Register
.add_plugins(RapierPhysicsPlugin::<MyHooks>::default())
```

See `references/advanced_collision_detection.md` for full hook API.

## Essential Properties Reference

```rust
// Rigid body components
GravityScale(0.5)           // 0 = no gravity, 2 = 2x
Damping { linear_damping: 0.5, angular_damping: 1.0 }
LockedAxes::ROTATION_LOCKED // Prevent tumbling
Dominance::group(10)        // Higher = immune to lower dominance contacts
Ccd::enabled()              // Continuous collision detection (fast objects)
Sleeping::disabled()        // Disable auto-sleep

// Collider components
Friction { coefficient: 0.7, combine_rule: CoefficientCombineRule::Average }
Restitution { coefficient: 0.5, combine_rule: CoefficientCombineRule::Average }
ColliderMassProperties::Density(2.0)
ActiveCollisionTypes::KINEMATIC_STATIC // Enable kinematic-vs-fixed collisions
```

## Best Practices

1. **Use SI units (meters)** - In 2D, use `pixels_per_meter()` for scaling
2. **Use density, not mass** - Let Rapier compute mass from collider shape
3. **Lock unused axes** - `LockedAxes::ROTATION_LOCKED` prevents character tumbling
4. **Use Sensors for triggers** - Not Dynamic bodies for hitboxes
5. **Kinematic for controlled movement** - Use KinematicPositionBased + character controller for players
6. **Collision groups for filtering** - More efficient than physics hooks for simple cases
7. **Build with optimizations** - Rapier is 100x slower without `opt-level = 3`

## Common Pitfalls

- **Body not falling:** Check it's Dynamic, has non-zero mass (collider density > 0), translations not locked
- **Force not working:** Body must be Dynamic with non-zero mass; force must be strong enough
- **Slow motion:** Using pixels as physics units. Use `pixels_per_meter()` in 2D
- **Simulation panic:** Two dynamic bodies with zero mass in contact produces NaN
- **Kinematic goes through walls:** Expected behavior. Use character controller for obstacle avoidance
- **Trimesh on dynamic body:** Objects get stuck (no interior). Use convex decomposition instead

## Performance

```toml
# Always optimize Rapier
[profile.dev.package.bevy_rapier3d]
opt-level = 3

# Max release performance
[profile.release]
codegen-units = 1
```

## References

- `references/getting_started.md` - Setup, cargo features, basic simulation, debug renderer
- `references/rigid_bodies.md` - Body types, position, velocity, forces, mass, damping, CCD, sleeping
- `references/colliders.md` - Shapes, sensors, friction, restitution, collision groups, active events/hooks
- `references/joints.md` - Fixed, revolute, prismatic, spherical joints and motors
- `references/joint_constraints.md` - MultibodyJointSet vs ImpulseJointSet, closed loops
- `references/character_controller.md` - Setup, slopes, autostep, snap-to-ground, filtering, collisions
- `references/scene_queries.md` - Ray cast, shape cast, point projection, intersection tests, filters
- `references/advanced_collision_detection.md` - Contact/intersection graphs, physics hooks, contact modification
- `references/multiple_contexts.md` - Multiple physics simulations, manual context creation
- `references/common_mistakes.md` - Performance, gravity, forces, panics, slow motion fixes
