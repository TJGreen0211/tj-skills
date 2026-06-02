# Getting Started - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/getting_started_bevy

## Overview

`bevy_rapier2d` and `bevy_rapier3d` crates integrate Rapier physics into Bevy using its plugin system.

## Cargo Dependencies

### 2D
```toml
[dependencies]
bevy = "*"
bevy_rapier2d = "*"
```

### 3D
```toml
[dependencies]
bevy = "*"
bevy_rapier3d = "*"
```

### With Features
```toml
# 2D with SIMD and debug rendering
bevy_rapier2d = { version = "*", features = ["simd-stable", "debug-render-2d"] }

# 3D with SIMD and debug rendering
bevy_rapier3d = { version = "*", features = ["simd-stable", "debug-render-3d"] }
```

## Available Features

- `debug-render-2d`/`debug-render-3d` - enables the debug-renderer plugin
- `simd-stable` - SIMD optimizations via `wide` crate (stable Rust)
- `simd-nightly` - SIMD optimizations via `packed_simd` crate (nightly Rust)
- `parallel` - parallelism via `rayon` crate
- `serde-serialize` - serialization of physics components with serde
- `enhanced-determinism` - cross-platform determinism (IEEE 754-2008)
- `wasm-bindgen` - WASM support

## Basic Simulation Example

### 2D
```rust
use bevy::prelude::*;
use bevy_rapier2d::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(RapierPhysicsPlugin::<NoUserData>::pixels_per_meter(100.0))
        .add_plugins(RapierDebugRenderPlugin::default())
        .add_systems(Startup, setup_graphics)
        .add_systems(Startup, setup_physics)
        .add_systems(Update, print_ball_altitude)
        .run();
}

fn setup_graphics(mut commands: Commands) {
    commands.spawn(Camera2d::default());
}

fn setup_physics(mut commands: Commands) {
    commands
        .spawn(Collider::cuboid(500.0, 50.0))
        .insert(Transform::from_xyz(0.0, -100.0, 0.0));

    commands
        .spawn(RigidBody::Dynamic)
        .insert(Collider::ball(50.0))
        .insert(Restitution::coefficient(0.7))
        .insert(Transform::from_xyz(0.0, 400.0, 0.0));
}

fn print_ball_altitude(positions: Query<&Transform, With<RigidBody>>) {
    for transform in positions.iter() {
        println!("Ball altitude: {}", transform.translation.y);
    }
}
```

### 3D
```rust
use bevy::prelude::*;
use bevy_rapier3d::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(RapierPhysicsPlugin::<NoUserData>::default())
        .add_plugins(RapierDebugRenderPlugin::default())
        .add_systems(Startup, setup_graphics)
        .add_systems(Startup, setup_physics)
        .add_systems(Update, print_ball_altitude)
        .run();
}

fn setup_graphics(mut commands: Commands) {
    commands.spawn((
        Camera3d::default(),
        Transform::from_xyz(-3.0, 3.0, 10.0).looking_at(Vec3::ZERO, Vec3::Y),
    ));
}

fn setup_physics(mut commands: Commands) {
    commands
        .spawn(Collider::cuboid(100.0, 0.1, 100.0))
        .insert(Transform::from_xyz(0.0, -2.0, 0.0));

    commands
        .spawn(RigidBody::Dynamic)
        .insert(Collider::ball(0.5))
        .insert(Restitution::coefficient(0.7))
        .insert(Transform::from_xyz(0.0, 4.0, 0.0));
}
```

## Debug Renderer

Enable by:
1. Enabling `debug-render-2d` or `debug-render-3d` cargo feature
2. Adding `RapierDebugRenderPlugin` to the Bevy app

## Key Notes

- Use `use bevy_rapier2d::prelude::*` or `use bevy_rapier3d::prelude::*` for common types
- In 2D, use `pixels_per_meter()` to scale between pixel and physics units
- In 3D, use SI units (meters) directly with `.default()`
