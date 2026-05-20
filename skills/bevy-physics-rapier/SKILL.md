---
created: 2026-05-19
modified: 2026-05-19
name: bevy-physics-rapier
description: "Bevy physics with Rapier: rigid bodies, colliders, joints, constraints, collision events, and physics hooks. Use when adding physics to Bevy games, working with bevy_rapier2d or bevy_rapier3d, or handling collisions, joints, and rigid body dynamics."
---

# Bevy Physics with Rapier

Expert knowledge for integrating the Rapier physics engine with Bevy via `bevy_rapier2d` or `bevy_rapier3d`.

## Quick Start

```rust
// Cargo.toml
// bevy_rapier2d = { version = "0.27", features = ["debug-render-2d"] }
// bevy_rapier3d = { version = "0.27", features = ["debug-render-3d"] }

use bevy::prelude::*;
use rapier2d::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(RapierPhysicsPlugin::<NoUserData>::default())
        .add_systems(Startup, spawn_player)
        .run();
}

fn spawn_player(mut commands: Commands) {
    commands.spawn((
        RigidBody::Dynamic,
        Transform::from_xyz(0.0, 5.0, 0.0),
        Collider::circle(1.0),
        LockedAxes::NONE,
    ));
    // Ground
    commands.spawn((
        RigidBody::Fixed,
        Collider::cuboid(10.0, 0.5),
    ));
}
```

## Core Concepts

### Rigid Body Types

| Type | Behavior | Use Case |
|------|----------|----------|
| `Dynamic` | Full physics simulation | Players, projectiles, debris |
| `KinematicPosition` | Moved by transform, pushes dynamics | Moving platforms, conveyors |
| `KinematicVelocity` | Moved by velocity, pushes dynamics | Scrolling backgrounds |
| `Fixed` | Immovable, infinite mass | Walls, ground, static obstacles |
| `Sensor` | Collision detection only, no response | Triggers, hitboxes, zones |

### Collider Shapes

```rust
// 2D shapes
Collider::circle(radius)
Collider::cuboid(half_width, half_height)
Collider::segment(p1, p2)          // Line segment (edge)
Collider::trapezoid(half_height, bottom, top)
Collider::polygon(vertices)
Collider::convex_hull(points)
Collider::composite                // From children with Collider bundles

// 3D shapes
Collider::sphere(radius)
Collider::cuboid(x, y, z)
Collider::capsule(half_height, radius)
Collider::cylinder(half_height, radius)
Collider::cone(half_height, radius)
Collider::convex_hull(points)
```

### Physics Material

```rust
PhysicsMaterial {
    restitution: 0.5,        // Bounciness (0 = no bounce, 1 = full bounce)
    friction: 0.8,           // Surface friction (0 = slippery, 1 = sticky)
    density: 1.0,            // Mass per unit area/volume (Dynamic only)
    friction_combine_rule: CoefficientCombineRule::Multiply,
    restitution_combine_rule: CoefficientCombineRule::Multiply,
    ..default()
}
```

## Collision Events

```rust
// Register in plugin setup
app.register_type::<CollisionEvent>()
   .add_event::<CollisionStarted>()
   .add_event::<CollisionEnded>();

#[derive(Event)]
struct CollisionStarted {
    entity_a: Entity,
    entity_b: Entity,
    manifold: Manifold,
}

// Listen for collisions
fn on_collision(
    mut events: EventReader<CollisionStarted>,
) {
    for event in events.read() {
        println!("Collision: {:?} vs {:?}", event.entity_a, event.entity_b);
    }
}
```

See `references/physics_patterns.md` for detailed patterns on joints, sensors, raycasting, velocity hooks, and more.

## Essential Commands

```bash
# Add 2D physics
cargo add bevy_rapier2d --features debug-render-2d

# Add 3D physics
cargo add bevy_rapier3d --features debug-render-3d

# Run with physics debug rendering
cargo run --features bevy_rapier2d/debug-render-2d
```

## Best Practices

1. **Use Sensors for triggers** - Don't use Dynamic bodies for hitboxes; use Sensors
2. **Lock unused axes** - Prevent unwanted rotation with `LockedAxes`
3. **Set gravity early** - Configure `Gravity` resource in your physics plugin
4. **Use density, not mass** - Let Rapier compute mass from collider shape
5. **Kinematic for controlled movement** - Use KinematicPosition for player characters you control directly
6. **Collision groups for filtering** - Use `CollisionGroups` to control what collides with what

## Common Pitfalls

- Forgetting to add `RapierPhysicsPlugin` causes silent physics failure
- `KinematicPosition` bodies must have their `Transform` mutated directly (not via velocity)
- Colliders are positioned relative to the `Transform` of the parent entity
- Sensor events fire every frame during overlap, not just on entry
- `LockedAxes::ROTATION` prevents tumbling for characters and boxes

## References

- `references/physics_patterns.md` - Joints, raycasting, velocity hooks, collision groups, gravity, timestep, and advanced patterns
