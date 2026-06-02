# Common Mistakes - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/common_mistakes

## Local Build Slower Than Demos

Rapier can be **100x slower** without optimizations.

```toml
# Build in release mode
cargo build --release

# Or optimize only Rapier in dev mode
[profile.dev.package.bevy_rapier3d]
opt-level = 3

# Further improve release performance (longer build times)
[profile.release]
codegen-units = 1
```

## Rigid Body Not Affected by Gravity

Check:
1. Gravity vector is non-zero
2. Body is Dynamic (not Fixed or Kinematic)
3. Translations are not locked
4. Body has non-zero mass

**Mass check:** If no collider attached, mass is zero unless set explicitly. With colliders, at least one must have non-zero density. Triangle mesh colliders don't auto-compute mass.

## Force Not Working

Check:
1. Body is Dynamic
2. Body has non-zero mass (or angular inertia for torques)
3. Force is strong enough (try 100,000 to verify)

## Simulation Panics

```
"assertion failed: proxy.aabb.maxs[dim] >= self.min_bound"
```

Caused by NaN values in collider positions. Most common cause: two dynamic rigid bodies with zero mass in contact. Fix by ensuring non-zero mass on all dynamic bodies.

## Everything Moving in Slow Motion

Using pixels as physics units makes objects huge relative to default gravity (-9.81 m/s^2).

**Solution:** Use SI units (meters). For 2D with pixel graphics, use `pixels_per_meter`:

```rust
// 1 physics meter = 50 pixels
RapierPhysicsPlugin::<NoUserData>::pixels_per_meter(50.0)
```

This auto-scales transforms, collider sizes, and positions between pixels and physics meters.
