# Multiple Physics Contexts - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/multiple_contexts

## Overview

Supports multiple independent physics simulations (e.g., per-level physics, parallel AI training).

## Default Context

On `PreStartup`, bevy_rapier spawns a default `RapierContext` entity marked with `DefaultRapierContext`. Use `ReadRapierContext` to access it.

## Disabling Automatic Context

```rust
RapierPhysicsPlugin::<NoUserData>::default()
    .with_custom_initialization(RapierContextInitialization::NoAutomaticRapierContext),
```

## Creating Manual Contexts

```rust
let mut context = commands.spawn(RapierContextSimulation::default());
```

## Context Linking

Entities managed by a `RapierContext` (colliders, joints, rigid bodies) have `RapierContextEntityLink` to identify their context.

## Shared Resources

All contexts share the same `TimestepMode` resource - they execute at the same time and rate.
