# Common Bevy Pitfalls Reference

## 1. Using Old Event System

**❌ Problem:**
```rust
// Bevy 0.15/0.16 event system doesn't work in 0.17+
#[derive(Event)]
struct MyEvent { data: String }

app.add_event::<MyEvent>()
   .add_systems(Update, handle_event);

fn handle_event(mut events: EventReader<MyEvent>) { /* ... */ }
fn trigger(mut events: EventWriter<MyEvent>) { /* ... */ }
```

**Symptoms:**
- Compilation error: `MyEvent is not a Message`
- `method 'send' not found for MessageWriter`
- `method 'read' not found for MessageReader`

**✅ Solution:**
Migrate to the observer pattern:
```rust
// Bevy 0.17/0.18 observer pattern
#[derive(Event, Clone)]  // Must derive Clone!
struct MyEvent { data: String }

app.add_observer(handle_event);  // Use observer, not system

fn handle_event(
    trigger: Trigger<MyEvent>,  // Trigger parameter
    // ... other params
) {
    let event = trigger.event();
}

fn trigger_event(mut commands: Commands) {
    commands.trigger(MyEvent { data: "test".into() });
}
```

See `references/bevy_specific_tips.md` for complete migration guide.

## 2. RenderTarget in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 pattern doesn't work in 0.18
commands.spawn((
    Camera3d::default(),
    Camera {
        target: RenderTarget::Image(image_handle.into()),
        ..default()
    },
));
```

**Symptoms:**
- `Camera` no longer has a `target` field
- `RenderTarget` is not found as expected

**✅ Solution:**
Use `RenderTarget` as a separate component:
```rust
// Bevy 0.18
commands.spawn((
    Camera3d::default(),
    RenderTarget::Image(image_handle.into()),
));
```

## 3. Querying Material Handles

**❌ Problem:**
```rust
// Bevy 0.15/0.16 pattern doesn't work in 0.17/0.18
Query<&Handle<StandardMaterial>>
```

**Symptoms:**
- `Handle<StandardMaterial> is not a Component`
- Query trait bounds not satisfied

**✅ Solution:**
Use the `MeshMaterial3d` wrapper:
```rust
Query<&MeshMaterial3d<StandardMaterial>>

// Access handle with .0
for material_3d in query.iter() {
    if let Some(material) = materials.get_mut(&material_3d.0) {
        material.emissive = color;
    }
}
```

## 4. Forgetting to Register Systems

**❌ Problem:**
```rust
// Created system but forgot to add to app
pub fn my_new_system() { /* ... */ }
```

**✅ Solution:**
Always add to `main.rs`:
```rust
.add_systems(Update, my_new_system)
```

## 5. Borrowing Conflicts

**❌ Problem:**
```rust
// Can't have multiple mutable borrows
mut query1: Query<&mut Transform>,
mut query2: Query<&mut Transform>,  // Error!
```

**✅ Solution:**
```rust
// Use get_many_mut for specific entities
mut query: Query<&mut Transform>,

if let Ok([mut a, mut b]) = query.get_many_mut([entity_a, entity_b]) {
    // Can mutate both
}
```

## 6. Infinite Loops with Events

**❌ Problem:**
```rust
// Observer triggers itself
fn handle_event(
    trigger: Trigger<MyEvent>,
    mut commands: Commands,
) {
    commands.trigger(MyEvent);  // Infinite loop!
}
```

**✅ Solution:**
Use different event types or add termination condition.

## 7. Not Using Changed<T>

**❌ Problem:**
```rust
// Runs every frame for every entity
fn system(query: Query<&BigFive>) {
    for traits in query.iter() {
        // Expensive calculation every frame
    }
}
```

**✅ Solution:**
```rust
// Only runs when BigFive changes
fn system(query: Query<&BigFive, Changed<BigFive>>) {
    for traits in query.iter() {
        // Only when needed
    }
}
```

## 8. Entity Queries After Despawn

**❌ Problem:**
```rust
commands.entity(entity).despawn();
// Later in same system
let component = query.get(entity).unwrap();  // Crash!
```

**✅ Solution:**
Commands apply at end of stage. Use `Ok()` pattern:
```rust
if let Ok(component) = query.get(entity) {
    // Safe
}
```

## 9. Material/Asset Handle Confusion

**❌ Problem:**
```rust
// Created material but didn't store handle
materials.add(StandardMaterial { .. });  // Handle dropped!
```

**✅ Solution:**
```rust
let material_handle = materials.add(StandardMaterial { .. });
commands.spawn((
    Mesh3d(mesh_handle),
    MeshMaterial3d(material_handle),
    // ...
));
```

## 10. System Ordering Issues

**❌ Problem:**
```rust
// UI updates before state changes
.add_systems(Update, (
    update_ui,
    process_input,  // Wrong order!
))
```

**✅ Solution:**
Order systems by dependencies:
```rust
.add_systems(Update, (
    // Input processing
    process_input,

    // State changes
    update_state,

    // UI updates (reads state)
    update_ui,
))
```

## 11. Not Filtering Queries Early

**❌ Problem:**
```rust
// Filter in loop (inefficient)
Query<(&A, Option<&B>, Option<&C>)>
// Then check in loop
```

**✅ Solution:**
```rust
// Filter in query (efficient)
Query<&A, (With<B>, Without<C>)>
```

## 12. BorderRadius in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 pattern - BorderRadius as separate component
commands.spawn((
    Node { /* ... */ },
    BorderRadius::all(Val::Px(8.0)),
));
```

**Symptoms:**
- `BorderRadius` is not a Component in 0.18

**✅ Solution:**
Use the `border_radius` field on `Node`:
```rust
// Bevy 0.18
commands.spawn((
    Node {
        border_radius: BorderRadius::all(Val::Px(8.0)),
        // ...
    },
));
```

## 13. LineHeight in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 - LineHeight was part of TextFont
TextFont {
    font_size: 24.0,
    line_height: Some(1.5),
    ..default()
}
```

**Symptoms:**
- `TextFont` no longer has a `line_height` field

**✅ Solution:**
Use `LineHeight` as a separate component:
```rust
// Bevy 0.18
commands.spawn((
    Text::new("Hello"),
    TextFont {
        font_size: 24.0,
        ..default()
    },
    LineHeight(1.5),
));
```

## 14. Using clear_children in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 method name
entity.clear_children();
entity.remove_child(child);
entity.clear_related::<Children>();
```

**Symptoms:**
- Method not found

**✅ Solution:**
Use the renamed `detach_*` methods:
```rust
// Bevy 0.18
entity.detach_all_children();
entity.detach_child(child);
entity.detach_all_related::<Children>();
```

## 15. AmbientLight in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 - AmbientLight as a resource or single component
commands.insert_resource(AmbientLight {
    color: Color::WHITE,
    brightness: 1.0,
});
```

**Symptoms:**
- `AmbientLight` is now a component, not a resource

**✅ Solution:**
Use `AmbientLight` as a component with `GlobalAmbientLight` resource:
```rust
// Bevy 0.18
commands.spawn(AmbientLight {
    color: Color::WHITE,
    brightness: 1.0,
});
```

## 16. Gizmos::cuboid Renamed in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17
gizmos.cuboid(transform, Color::RED);
```

**✅ Solution:**
```rust
// Bevy 0.18
gizmos.cube(transform, Color::RED);
```

## 17. #[reflect(...)] Syntax in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 - brackets or braces
#[derive(Clone, Reflect)]
#[reflect[Clone]]

#[derive(Clone, Reflect)]
#[reflect{Clone}]
```

**Symptoms:**
- Compilation error with non-parentheses syntax

**✅ Solution:**
Only use parentheses:
```rust
// Bevy 0.18
#[derive(Clone, Reflect)]
#[reflect(Clone)]
```

## 18. Feature Flags in Bevy 0.18

**❌ Problem:**
```toml
# Bevy 0.17
bevy = { version = "0.17", default-features = false, features = ["animation"] }
```

**Symptoms:**
- `animation` feature not found
- Input features not available

**✅ Solution:**
Use updated feature names:
```toml
# Bevy 0.18
bevy = { version = "0.18", default-features = false, features = [
  "gltf_animation",     # renamed from "animation"
  "mouse",
  "keyboard",
  "gamepad",
  "touch",
  "gestures",
] }

# Or use feature collections:
bevy = { version = "0.18", features = ["3d", "render"] }
```

## 19. Material Plugin in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 - plugin fields
MaterialPlugin::<MyMaterial> {
    prepass_enabled: false,
    shadows_enabled: false,
}
```

**✅ Solution:**
Use `Material` trait methods:
```rust
// Bevy 0.18
impl Material for MyMaterial {
    // ...

    fn enable_prepass() -> bool { false }
    fn enable_shadows() -> bool { false }
}
```

## 20. Mesh Functions in Bevy 0.18

**❌ Problem:**
```rust
// Bevy 0.17 - direct mesh access
let mesh = assets.get(handle).unwrap();
mesh.attribute(Mesh::ATTRIBUTE_POSITION);
mesh.insert_attribute(...);
```

**Symptoms:**
- Panic when accessing `RENDER_WORLD`-only meshes

**✅ Solution:**
Use `try_*` methods for fallible access:
```rust
// Bevy 0.18
let mesh = assets.get(handle).unwrap();
let positions = mesh.try_attribute(Mesh::ATTRIBUTE_POSITION);
mesh.try_insert_attribute(...);
```
