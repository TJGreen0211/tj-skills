# Character Controller - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/character_controller

## Overview

The Kinematic Character Controller handles obstacle avoidance for kinematic bodies using move-and-slide. Supports:
- Stopping at obstacles
- Sliding on slopes
- Climbing stairs automatically
- Walking over small obstacles
- Interacting with moving platforms

**Note:** Does NOT support rotational movement. Only translations.

## Setup

```rust
fn setup_physics(mut commands: Commands) {
    commands
        .spawn(RigidBody::KinematicPositionBased)
        .insert(Collider::ball(0.5))
        .insert(KinematicCharacterController::default());
}

fn update_system(mut controllers: Query<&mut KinematicCharacterController>) {
    for mut controller in controllers.iter_mut() {
        controller.translation = Some(Vec3::new(1.0, -5.0, -1.0) * time.delta_secs());
    }
}

fn read_result_system(controllers: Query<(Entity, &KinematicCharacterControllerOutput)>) {
    for (entity, output) in controllers.iter() {
        println!("Moved by {:?}, grounded: {:?}",
            output.effective_translation, output.grounded);
    }
}
```

**Recommended shapes:** cuboid, ball, or capsule (fewer computations, less numerical issues).

## Character Offset

Small gap between character and environment for stability. Default is auto-computed.

```rust
KinematicCharacterController {
    offset: CharacterLength::Absolute(0.01),
    // or
    offset: CharacterLength::Relative(0.01),  // relative to collider height
    ..default()
}
```

**Warning:** Do not change offset after creation.

## Up Vector

Default: positive Y axis. Controls what direction is "vertical".

```rust
KinematicCharacterController {
    up: Vec3::Y,  // default
    // or Vec3::X for side-scrollers
    ..default()
}
```

## Slopes

```rust
KinematicCharacterController {
    max_slope_climb_angle: 45_f32.to_radians(),  // max climbable slope
    min_slope_slide_angle: 30_f32.to_radians(),  // when to slide down
    ..default()
}
```

## Autostep (Stairs and Small Obstacles)

```rust
KinematicCharacterController {
    autostep: Some(CharacterAutostep {
        max_height: CharacterLength::Absolute(0.5),
        min_width: CharacterLength::Absolute(0.2),
        include_dynamic_bodies: true,
    }),
    ..default()
}
```

Only activates when character is touching the ground before the obstacle.

## Snap-to-Ground

For going downstairs or staying on downhill surfaces:

```rust
KinematicCharacterController {
    snap_to_ground: Some(CharacterLength::Absolute(0.5)),
    // or CharacterLength::Relative(0.2) for 20% of character height
    ..default()
}
```

## Filtering

```rust
KinematicCharacterController {
    filter_flags: QueryFilterFlags::EMPTY,  // exclude families
    filter_groups: CollisionGroups::new(...),  // collision group filtering
    ..default()
}
```

## Collisions and Dynamic Body Impulses

```rust
// Read collisions from output
for collision in &output.collisions {
    // Process collision (chronological order)
}

// Auto-apply impulses to dynamic bodies hit by character
KinematicCharacterController {
    apply_impulse_to_dynamic_bodies: true,
    ..default()
}
```

## Gravity

Not automatic. Manually add downward component to `controller.translation` each frame.
