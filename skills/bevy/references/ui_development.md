# Bevy UI Development Reference

## Bevy UI Hierarchy

Bevy UI uses a flexbox-like layout system:

```rust
commands
    .spawn((
        Node {
            position_type: PositionType::Absolute,
            left: Val::Px(10.0),
            top: Val::Px(10.0),
            width: Val::Px(300.0),
            padding: UiRect::all(Val::Px(10.0)),
            flex_direction: FlexDirection::Column,
            border_radius: BorderRadius::all(Val::Px(8.0)),  // 0.18: part of Node
            ..default()
        },
        BackgroundColor(Color::srgba(0.1, 0.1, 0.1, 0.9)),
    ))
    .with_children(|parent| {
        parent.spawn((
            Text::new("Title"),
            TextFont { font_size: 16.0, ..default() },
            LineHeight(1.5),  // 0.18: separate component
            TextColor(Color::WHITE),
        ));
    });
```

## UI Component Pattern

**1. Marker Components for UI Elements**
```rust
#[derive(Component)]
pub struct SpellBar;

#[derive(Component)]
pub struct HoverTooltip;

#[derive(Component)]
pub struct InspectPanel;
```

**2. Setup System (Startup)**
```rust
pub fn setup_ui(mut commands: Commands) {
    commands.spawn((
        SpellBar,
        Node { /* layout */ },
        BackgroundColor(/* color */),
    ))
    .with_children(|parent| {
        // Child elements
    });
}
```

**3. Update System (Update)**
```rust
pub fn update_ui(
    state: Res<GameState>,
    mut query: Query<&mut Text, With<SpellBar>>,
) {
    for mut text in query.iter_mut() {
        **text = format!("State: {:?}", state);
    }
}
```

## UI Best Practices

### Layout Tips

- Use `Val::Px()` for fixed sizes
- Use `Val::Percent()` for responsive layouts
- Use `flex_direction: FlexDirection::Column` for vertical stacking
- Use `flex_direction: FlexDirection::Row` for horizontal stacking
- Use `justify_content` and `align_items` for alignment

### Positioning

**Absolute positioning (HUD elements):**
```rust
Node {
    position_type: PositionType::Absolute,
    left: Val::Px(10.0),
    top: Val::Px(10.0),
    ..default()
}
```

**Centered element:**
```rust
Node {
    position_type: PositionType::Absolute,
    left: Val::Percent(50.0),
    top: Val::Percent(50.0),
    margin: UiRect {
        left: Val::Px(-150.0),  // Half of width
        top: Val::Px(-100.0),   // Half of height
        ..default()
    },
    width: Val::Px(300.0),
    height: Val::Px(200.0),
    ..default()
}
```

### Border Radius (Bevy 0.18)

In Bevy 0.18, `BorderRadius` is a field on `Node`, not a separate component:

```rust
// ✅ Bevy 0.18
Node {
    border_radius: BorderRadius::all(Val::Px(8.0)),
    // or individual sides:
    // border_radius: BorderRadius {
    //     top_left: Val::Px(8.0),
    //     top_right: Val::Px(8.0),
    //     bottom_left: Val::Px(0.0),
    //     bottom_right: Val::Px(0.0),
    // },
    ..default()
}
```

### Visibility Control

```rust
// Show/hide with Display
mut node: Query<&mut Node, With<Panel>>

// Hide
node.display = Display::None;

// Show
node.display = Display::Flex;
```

### Color and Styling

```rust
// Background
BackgroundColor(Color::srgba(0.1, 0.1, 0.1, 0.9))

// Border
BorderColor::all(Color::srgba(0.3, 0.6, 0.9, 1.0))

// Highlight on selection
*bg_color = BackgroundColor(Color::srgba(0.2, 0.4, 0.6, 1.0));
*border_color = BorderColor::all(Color::srgba(0.3, 0.6, 0.9, 1.0));
```

### Text and Line Height (Bevy 0.18)

In Bevy 0.18, `LineHeight` is a separate component:

```rust
// ✅ Bevy 0.18
commands.spawn((
    Text::new("Hello World"),
    TextFont {
        font_size: 24.0,
        font: asset_server.load("fonts/font.ttf"),
        ..default()
    },
    TextColor(Color::WHITE),
    LineHeight(1.5),  // Separate component in 0.18
));

// ❌ Bevy 0.17 - line_height was part of TextFont
TextFont {
    font_size: 24.0,
    line_height: Some(1.5),  // No longer exists
    ..default()
}
```

### Text Updates

```rust
// Update text content
**text = "New content".to_string();

// Or with formatting
**text = format!("Value: {:.2}", value);

// Multi-line text
**text = "Line 1\nLine 2\nLine 3".to_string();
```

### BorderRect (Bevy 0.18)

In Bevy 0.18, `BorderRect` uses `Vec2` fields instead of directional `Val`:

```rust
// ✅ Bevy 0.18
BorderRect {
    min_inset: Vec2::new(10.0, 20.0),  // left, top
    max_inset: Vec2::new(10.0, 20.0),  // right, bottom
}

// ❌ Bevy 0.17 - individual Val fields
BorderRect {
    left: Val::Px(10.0),
    top: Val::Px(20.0),
    right: Val::Px(10.0),
    bottom: Val::Px(20.0),
}
```
