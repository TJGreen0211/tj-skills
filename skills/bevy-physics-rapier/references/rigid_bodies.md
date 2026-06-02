# Rigid Bodies - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/rigid_bodies

## Overview

Rigid bodies handle dynamics and kinematics of solids. Colliders must be attached for collision detection.

## Creation

```rust
commands
    .spawn(RigidBody::Dynamic)
    .insert(Transform::from_xyz(0.0, 5.0, 0.0))
    .insert(Velocity { linvel: Vec2::new(1.0, 2.0), angvel: 0.2 })
    .insert(GravityScale(0.5))
    .insert(Sleeping::disabled())
    .insert(Ccd::enabled());
```

## Rigid Body Types

| Type | Description | Use Case |
|------|-------------|----------|
| `RigidBody::Dynamic` | Affected by forces and contacts | Players, projectiles, debris |
| `RigidBody::Fixed` | Immovable, infinite mass | Ground, walls, static obstacles |
| `RigidBody::KinematicPositionBased` | Position controlled by user; velocity deduced | Moving platforms, elevators |
| `RigidBody::KinematicVelocityBased` | Velocity controlled by user; position deduced | Moving platforms, conveyors |

**Important:** Kinematic bodies ignore all contact forces. They go through walls. Use scene queries or character controller for obstacle avoidance.

## Position

Stored in Bevy `Transform` component. Directly changing position = teleporting (non-physical).

- **Dynamic bodies:** Prefer forces, impulses, or velocity modification
- **KinematicPositionBased:** Modify `Transform` directly
- **KinematicVelocityBased:** Set velocity instead of position

```rust
fn modify_body_translation(mut positions: Query<&mut Transform, With<RigidBody>>) {
    for mut position in positions.iter_mut() {
        position.translation.y += 0.1;
    }
}
```

## Velocity

Only relevant for **dynamic** bodies. Applied at center-of-mass.

```rust
// 2D
commands.spawn(RigidBody::Dynamic).insert(Velocity {
    linvel: Vec2::new(0.0, 2.0),
    angvel: 0.4,
});

// 3D
commands.spawn(RigidBody::Dynamic).insert(Velocity {
    linvel: Vec3::new(0.0, 2.0, 0.0),
    angvel: Vec3::new(0.2, 0.4, 0.8),
});
```

## Gravity

Global gravity: `RapierConfiguration::gravity`. Per-body scale: `GravityScale`.

```rust
commands.spawn(RigidBody::Dynamic).insert(GravityScale(2.0));
// 0.0 = no gravity, 2.0 = 2x gravity, negative = flipped direction
```

## Forces and Impulses

Forces affect acceleration; impulses affect velocity directly. Forces are persistent across steps.

```rust
// 2D
commands
    .spawn(RigidBody::Dynamic)
    .insert(ExternalForce { force: Vec2::new(1000.0, 2000.0), torque: 140.0 })
    .insert(ExternalImpulse { impulse: Vec2::new(100.0, 200.0), torque_impulse: 14.0 });

// 3D
commands
    .spawn(RigidBody::Dynamic)
    .insert(ExternalForce { force: Vec3::new(10.0, 20.0, 30.0), torque: Vec3::new(1.0, 2.0, 3.0) })
    .insert(ExternalImpulse { impulse: Vec3::new(1.0, 2.0, 3.0), torque_impulse: Vec3::new(0.1, 0.2, 0.3) });
```

**Troubleshooting forces not working:**
1. Body must be Dynamic
2. Body must have non-zero mass (check collider density)
3. Force must be strong enough

## Mass Properties

Auto-computed from attached colliders (shape + density). Can set manually:

```rust
// Auto from collider density
commands
    .spawn(RigidBody::Dynamic)
    .insert(Collider::ball(1.0))
    .insert(ColliderMassProperties::Density(2.0));

// Manual additional mass
commands
    .spawn(RigidBody::Dynamic)
    .insert(AdditionalMassProperties::Mass(10.0));
```

**Note:** Zero mass = infinite mass (immune to linear forces). Zero angular inertia = immune to torques.

## Locking Translations/Rotations

```rust
// 2D
commands.spawn(RigidBody::Dynamic).insert(LockedAxes::TRANSLATION_LOCKED);
commands.spawn(RigidBody::Dynamic).insert(LockedAxes::ROTATION_LOCKED);

// 3D
commands.spawn(RigidBody::Dynamic)
    .insert(LockedAxes::TRANSLATION_LOCKED | LockedAxes::ROTATION_LOCKED_X);
```

## Damping

```rust
commands.spawn(RigidBody::Dynamic).insert(Damping {
    linear_damping: 0.5,
    angular_damping: 1.0,
});
```

## Dominance

Groups in `[-127, 127]` (default 0). Higher dominance = immune to contact forces from lower dominance bodies.

```rust
commands.spawn(RigidBody::Dynamic).insert(Dominance::group(10));
```

## Continuous Collision Detection (CCD)

Prevents tunneling for fast-moving objects. Only enable on fast dynamic bodies.

```rust
commands.spawn(RigidBody::Dynamic).insert(Ccd::enabled());
```

## Sleeping

Bodies at rest auto-sleep to save CPU. Auto-woken on interaction. Components modified = auto-wake (except gravity changes).

```rust
commands.spawn(RigidBody::Dynamic).insert(Sleeping::disabled());
```
