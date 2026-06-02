# Scene Queries - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/scene_queries

## Overview

Geometric queries against all colliders via `RapierContext` (accessed through `ReadRapierContext`).

## Ray Casting

```rust
fn cast_ray(rapier_context: ReadRapierContext) {
    let ctx = rapier_context.single().unwrap();
    let ray_pos = Vec3::new(1.0, 2.0, 3.0);
    let ray_dir = Vec3::new(0.0, 1.0, 0.0);
    let max_toi = 4.0;
    let solid = true;
    let filter = QueryFilter::default();

    // First hit (distance only)
    if let Some((entity, toi)) = ctx.cast_ray(ray_pos, ray_dir, max_toi, solid, filter) {
        let hit_point = ray_pos + ray_dir * toi;
    }

    // First hit with normal
    if let Some((entity, intersection)) = ctx.cast_ray_and_get_normal(ray_pos, ray_dir, max_toi, solid, filter) {
        let hit_point = intersection.point;
        let hit_normal = intersection.normal;
    }

    // All hits (callback)
    ctx.intersections_with_ray(ray_pos, ray_dir, max_toi, solid, filter, |entity, intersection| {
        true  // return false to stop searching
    });
}
```

Parameters:
- `max_toi` - maximum ray length (time-of-impact)
- `solid` - if true, ray starting inside shape reports hit at origin; if false, reports exit point

## Shape Casting (Sweep Tests)

```rust
fn cast_shape(rapier_context: ReadRapierContext) {
    let ctx = rapier_context.single().unwrap();
    let shape = Collider::cuboid(1.0, 2.0, 3.0);
    let shape_pos = Vec3::new(1.0, 2.0, 3.0);
    let shape_rot = Quat::from_rotation_z(0.8);
    let shape_vel = Vec3::new(0.1, 0.4, 0.2);
    let filter = QueryFilter::default();
    let options = ShapeCastOptions {
        max_time_of_impact: 4.0,
        target_distance: 0.0,
        stop_at_penetration: false,
        compute_impact_geometry_on_penetration: false,
    };

    if let Some((entity, hit)) = ctx.cast_shape(shape_pos, shape_rot, shape_vel, &shape, options, filter) {
        // hit.toi, hit.witness1, hit.witness2, hit.normal1, hit.normal2
    }
}
```

## Point Projection

```rust
// Closest collider to point
if let Some((entity, projection)) = ctx.project_point(point, solid, filter) {
    println!("Projection: {}, inside: {}", projection.point, projection.is_inside);
}

// All colliders containing point
ctx.intersections_with_point(point, filter, |entity| { true });
```

## Intersection Tests

```rust
// Exact: all colliders intersecting a shape
ctx.intersections_with_shape(shape_pos, shape_rot, &shape, filter, |entity| { true });

// Approximate: all colliders whose AABB intersects given AABB
let aabb = Aabb3d::new(Vec3::new(-1.0, -2.0, -3.0), Vec3::new(1.0, 2.0, 3.0));
ctx.colliders_with_aabb_intersecting_aabb(aabb, |entity| { true });
```

## Query Filters

```rust
let predicate = |handle| {
    custom_data_query.get(handle).is_ok_and(|d| d.data == 10)
};

let filter = QueryFilter::exclude_dynamic()
    .exclude_sensors()
    .exclude_rigid_body(player_handle)
    .groups(CollisionGroups::new(Group::GROUP_1 | Group::GROUP_2, Group::GROUP_1))
    .predicate(&predicate);
```

Filter options:
- `flags` - exclude by type (dynamic, sensors, etc.)
- `groups` - collision group filtering
- `exclude_collider` / `exclude_rigid_body` - exclude specific entities
- `predicate` - custom closure for arbitrary filtering
