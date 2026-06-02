# Joints - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/joints

## Overview

Joints restrict relative motion between rigid bodies. Inserted into `ImpulseJointSet` or `MultibodyJointSet`.

## Joint Types

| Joint | 2D DOF | 3D DOF | Description |
|-------|--------|--------|-------------|
| Fixed | None | None | Bodies don't move relative to each other |
| Revolute | 1 rotation | 1 rotation | Hinge/wheel (axis specified) |
| Prismatic | 1 translation | 1 translation | Slider (axis specified, supports limits) |
| Spherical | 1 rotation | 3 rotations | Ball-in-socket (ragdoll arms, pendulums) |
| Generic | Configurable | Configurable | Free, Cartesian, Planar, Cylindrical, Pin-slot, Rectangular, Universal |

## Fixed Joint

```rust
// 2D
let joint = FixedJointBuilder::new()
    .local_anchor1(Vec2::new(0.0, -20.0));
commands
    .spawn(RigidBody::Dynamic)
    .insert(Collider::cuboid(5.0, 5.0))
    .insert(ImpulseJoint::new(parent_entity, joint));

// 3D
let joint = FixedJointBuilder::new()
    .local_anchor1(Vec3::new(0.0, 0.0, -2.0));
```

**Note:** Attaching multiple colliders to one rigid body is more efficient than using fixed joints.

## Spherical Joint

Prevents relative translation at anchor points. Allows free rotation.

```rust
let joint = SphericalJointBuilder::new()
    .local_anchor1(Vec3::new(0.0, 0.0, 1.0))
    .local_anchor2(Vec3::new(0.0, 0.0, -3.0));
commands
    .spawn(RigidBody::Dynamic)
    .insert(ImpulseJoint::new(parent_entity, joint));
```

**Note:** In 2D, use `RevoluteJoint` instead (they're equivalent).

## Revolute Joint

Allows rotation along one axis only.

```rust
// 2D
let joint = RevoluteJointBuilder::new()
    .local_anchor1(Vec2::new(0.0, 1.0))
    .local_anchor2(Vec2::new(0.0, -5.0));

// 3D (axis matters)
let joint = RevoluteJointBuilder::new(Vec3::X)
    .local_anchor1(Vec3::new(0.0, 0.0, 1.0))
    .local_anchor2(Vec3::new(0.0, 0.0, -3.0));
```

## Prismatic Joint

Allows translation along one axis. Supports limits.

```rust
let joint = PrismaticJointBuilder::new(Vec3::X)
    .local_anchor1(Vec3::new(0.0, 0.0, 1.0))
    .local_anchor2(Vec3::new(0.0, 0.0, -3.0))
    .limits([-2.0, 5.0]);  // Signed distance range along axis
```

## Joint Motors

Spherical, revolute, and prismatic joints support PD controller motors.

```rust
// Configure motor with target velocity
let joint = PrismaticJointBuilder::new(Vec3::X)
    .motor_velocity(0.1, 0.05);  // (target_vel, damping)

// Methods available on joint builders:
// .configure_motor_position(target_pos, stiffness, damping)
// .configure_motor_velocity(target_vel, damping)
// .configure_motor(target_pos, target_vel, stiffness, damping)
// .configure_motor_model(model)
```
