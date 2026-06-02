# Colliders - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/colliders

## Overview

Colliders define geometric shapes for contact generation and collision events.

## Creation and Attachment

```rust
// Single collider on same entity as RigidBody
commands
    .spawn(RigidBody::Dynamic)
    .insert(Collider::ball(0.5));

// Multiple colliders as children (position relative to parent)
commands
    .spawn((RigidBody::Dynamic, GlobalTransform::default()))
    .with_children(|children| {
        children.spawn(Collider::ball(0.5))
            .insert(Transform::from_xyz(0.0, 0.0, -1.0));
        children.spawn(Collider::ball(0.5))
            .insert(Transform::from_xyz(0.0, 0.0, 1.0));
    });

// Standalone collider (no RigidBody = Fixed implicitly)
commands
    .spawn(Collider::cuboid(1.0, 2.0))
    .insert(Transform::from_xyz(2.0, 0.0, 0.0));
```

## Collider Types

- **Solid collider** (default) - generates contact forces
- **Sensor** - only generates intersection events, no contact forces

```rust
commands.spawn(Collider::ball(0.5)).insert(Sensor);
```

## Shapes

### Basic Shapes

```rust
// 2D
Collider::ball(radius)
Collider::cuboid(half_width, half_height)
Collider::capsule(half_height, radius)
Collider::segment(p1, p2)

// 3D
Collider::ball(radius)
Collider::cuboid(half_x, half_y, half_z)
Collider::capsule(half_height, radius)
Collider::cylinder(half_height, radius)
Collider::cone(half_height, radius)
```

### Convex Meshes
```rust
Collider::convex_hull(points)           // Auto-computes convex hull
Collider::convex_mesh(points, indices)  // 3D, assumes already convex
Collider::convex_polyline(points)       // 2D, assumes already convex
```

### Triangle Meshes (3D) / Polylines (2D)
```rust
Collider::trimesh(vertices, indices)
Collider::trimesh_with_flags(vertices, indices, flags)
Collider::polyline(vertices, indices)
```
**Warning:** Discouraged for dynamic bodies (no interior, objects get stuck). Use convex decomposition instead.

### Heightfields
```rust
// 3D: X-Z plane grid with Y heights
Collider::heightfield(heights_matrix, scale)
// 2D: X axis with Y heights
Collider::heightfield(heights_vector, scale)
```

### Voxels
```rust
Collider::voxels_from_points(cell_size, points)
```

### Compound Shapes (Convex Decomposition)
```rust
Collider::compound(vec![
    (pos1, rot1, shape1),
    (pos2, rot2, shape2),
])
Collider::convex_decomposition(vertices, indices)  // Auto VHACD decomposition
```

### Round Shapes
```rust
Collider::round_cuboid(half_x, half_y, half_z, border_radius)
Collider::round_cylinder(half_height, radius, border_radius)
Collider::round_cone(half_height, radius, border_radius)
```

## Mass Properties

```rust
// By density (recommended)
ColliderMassProperties::Density(2.0)

// By mass
ColliderMassProperties::Mass(0.8)

// Explicit
ColliderMassProperties::MassProperties(MassProperties {
    local_center_of_mass: Vec3::new(0.0, 1.0, 2.0),
    mass: 0.5,
    principal_inertia_local_frame: Quat::IDENTITY,
    principal_inertia: Vec3::new(0.3, 0.4, 0.5),
})
```

## Position

Transform is relative to parent rigid body when attached as child.

## Friction

```rust
commands.spawn(Collider::ball(0.5)).insert(Friction {
    coefficient: 0.7,
    combine_rule: CoefficientCombineRule::Min,  // Average, Min, Multiply, Max
});
```

Combine rule precedence: Max > Multiply > Min > Average (default: Average)

## Restitution

```rust
commands.spawn(Collider::ball(0.5)).insert(Restitution {
    coefficient: 0.7,  // 0 = no bounce, 1 = full bounce
    combine_rule: CoefficientCombineRule::Min,
});
```

## Collision Groups and Solver Groups

32 groups (u32 bitmasks). Membership = what groups collider belongs to. Filter = what groups it interacts with.

```rust
commands
    .spawn(Collider::ball(0.5))
    .insert(CollisionGroups::new(
        Group::GROUP_1 | Group::GROUP_3 | Group::GROUP_4,  // membership
        Group::GROUP_3,                                     // filter
    ))
    .insert(SolverGroups::new(
        Group::GROUP_1 | Group::GROUP_2,                    // membership
        Group::GROUP_1 | Group::GROUP_2 | Group::GROUP_4,   // filter
    ));
```

- `CollisionGroups` - filters narrow-phase contact computation (preferred, skips more)
- `SolverGroups` - filters contact force computation only (use when you need contacts but not forces)

Bitwise check for interaction:
```
(A.memberships & B.filter) != 0 && (B.memberships & A.filter) != 0
```

## Active Collision Types

Enable collisions between normally-disabled pairs (e.g., kinematic vs fixed):

```rust
commands.spawn(Collider::ball(0.5))
    .insert(ActiveCollisionTypes::default() | ActiveCollisionTypes::KINEMATIC_STATIC);
```

## Active Events

Enable collision events:

```rust
commands.spawn(Collider::ball(0.5))
    .insert(ActiveEvents::COLLISION_EVENTS);
```

## Active Hooks

Enable physics hooks for custom filtering/modification:

```rust
commands.spawn(Collider::ball(0.5))
    .insert(ActiveHooks::FILTER_CONTACT_PAIRS | ActiveHooks::MODIFY_SOLVER_CONTACTS);
```
