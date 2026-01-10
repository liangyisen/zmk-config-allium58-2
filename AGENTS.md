# AGENTS.md - ZMK Configuration for Allium58

ZMK firmware configuration for the Allium58 keyboard (Lily58 split layout).

## Build Commands

### Local Build with West

```bash
# Initialize west workspace (first time only)
west init -l config
west update

# Build left half with OLED
west build --board nice_nano_v2 -- -DSHIELD="lily58_left nice_oled"

# Build right half with OLED
west build --board nice_nano_v2 -- -DSHIELD="lily58_right nice_oled"

# Build settings reset shield
west build --board nice_nano_v2 -- -DSHIELD="settings_reset"
```

### GitHub Actions

Pushes/PRs trigger builds via `build.yaml` matrix. Outputs: `build/zephyr/zmk.uf2`

## Code Style Guidelines

### Keymap Files (`.keymap`)

**Includes:**
```dts
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/pointing.h>
```

**Layer Definitions:**
- Named `default_layer`, `lower_layer`, `raise_layer`, `adjust_layer`, `extra`
- Include `display-name` property
- Use `//` comments for layout diagrams

**Behavior References:**
- `&kp` - Key press
- `&mo` - Momentary layer switch
- `&to` - Toggle layer
- `&mt` - Hold-tap (mod-tap)
- `&trans` - Transparent
- `&bt` - Bluetooth
- `&msc` - Mouse scroll
- `&mmv` - Mouse move
- `&mkp` - Mouse click

**Formatting:**
- Vertically align bindings arrays with tabs
- Use `&kp KEY_NAME` format

### Configuration Files (`.conf`)

```conf
CONFIG_FEATURE_NAME=y
CONFIG_ZMK_KEYBOARD_NAME="name"
```

- Comment unused with `#`
- Group related settings with blank lines
- Alphabetize settings within groups

### Device Tree Overlays (`.overlay`)

**GPIO Pin Configuration:**
```dts
row-gpios = <&pro_micro 5 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>;
```

**Encoder Configuration:**
```dts
compatible = "alps,ec11";
a-gpios = <&pro_micro 21 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;
steps = <80>;
```

### Combo Definitions

```dts
combos {
    compatible = "zmk,combos";

    combo_name {
        bindings = <&kp KEY>;
        key-positions = <0 1>;
        layers = <0>;
    };
};
```

### Conditional Layers

```dts
conditional_layers {
    compatible = "zmk,conditional-layers";

    tri_layer {
        if-layers = <1 2>;
        then-layer = <3>;
    };
};
```

### Naming Conventions

- **Layers**: `default_layer`, `lower_layer`, `raise_layer`, `adjust_layer`, `extra`
- **Behaviors**: `to_layer_2_binding` (underscores)
- **Combo names**: snake_case
- **Key positions**: 0-indexed numeric

### Import Order

1. System includes
2. Layer defines (`#define`)
3. Root node with conditional layers
4. Combos node
5. Behaviors node
6. Keymap node with layer definitions

### Common Key Codes

| Code | Description |
|------|-------------|
| `ESC`, `TAB`, `BSPC` | Modifier keys |
| `N1`-`N0` | Number row |
| `A`-`Z` | Letters |
| `SEMI`, `SQT`, `COMM`, `DOT`, `FSLH` | Punctuation |
| `LEFT_SHIFT`, `LCTRL`, `LALT`, `LGUI` | Modifiers |
| `GRAVE`, `MINUS`, `EQUAL`, `LBRC`, `RBCK`, `BSLH` | Symbols |

### Bluetooth

- `&bt BT_CLR` - Clear all pairings
- `&bt BT_SEL n` - Select profile (0-4)

### Testing

Manual testing only. Flash firmware and test keypresses. Use ZMK Studio for runtime adjustments.

## Project Structure

```
├── build.yaml              # GitHub Actions build matrix
├── config/
│   ├── lily58.conf         # Kconfig settings
│   ├── lily58.keymap       # Main keymap
│   └── west.yml            # West manifest
├── boards/shields/lily58/
│   ├── lily58.keymap       # Shield keymap
│   ├── lily58.dtsi         # Hardware definition
│   ├── lily58_left.overlay
│   ├── lily58_right.overlay
│   └── Kconfig.*           # Shield Kconfig
└── .zmk/                   # Pinned ZMK and modules
```

## Notes for Agents

- Keyboard firmware config, not application code
- Build requires Zephyr SDK and west toolchain
- Keymap changes require rebuild and flash
- GitHub Actions auto-builds on push
- Do not modify `.zmk/` submodule contents
