# Macro & Controller Software Reference

> ESP32-only software feature reference.  
> This document defines the available controller functionality, macro keywords, variables, inputs, outputs, modes, layouts, layers, and configuration concepts.

---

# 1. Macro System

Macros are small programs executed by the controller's non-blocking macro VM.

A macro can react to an input event, perform logic, manipulate variables, send HID commands, change layers, control LEDs, and transmit ESP-NOW data.

## 1.1 Macro Triggers

### `ON_PRESS`

Runs a macro when an input changes from released to pressed.

```text
ON_PRESS
    TAP KEY_A
END
```

Useful for normal button actions.

---

### `ON_RELEASE`

Runs a macro when an input changes from pressed to released.

```text
ON_RELEASE
    RELEASE KEY_SHIFT
END
```

Useful for actions that must happen when a control is released.

---

### `ON_HOLD`

Runs or activates behavior while an input remains held.

Useful for modifier-style controls, temporary layers, and continuous actions.

---

### `ON_REPEAT`

Runs repeatedly while an input is held.

The repeat interval is configurable by the input/macro system.

Useful for:

- repeated keyboard input
- continuous scrolling
- repeated commands

---

### `ON_LONG_PRESS`

Runs after an input has remained pressed for a configured duration.

Useful for secondary actions such as:

- opening configuration mode
- switching profiles
- resetting a function

---

### Gesture / Zone Trigger

Allows a touchpad region or gesture to trigger a macro.

Examples:

- entering a trackpad zone
- leaving a zone
- tapping a zone
- swiping
- circular motion
- gesture completion

---

# 2. Control Flow Keywords

Control-flow commands determine how a macro executes.

## `LOOP [count]`

Repeats a block a fixed number of times.

```text
LOOP 5
    TAP KEY_A
END
```

The example presses `A` five times.

---

## `LOOP WHILE [condition]`

Repeats a block while a condition remains true.

```text
LOOP WHILE SYS_JOY_X > 500
    MOUSE_MOVE_REL 5 0
END
```

The loop stops when the condition becomes false.

The VM must enforce an execution limit so a condition that never becomes false cannot lock the controller.

---

## `IF [condition]`

Executes a block only when a condition is true.

```text
IF SYS_JOY_X > 500
    MOUSE_MOVE_REL 10 0
END
```

---

## `ELSE`

Executes an alternative block when the preceding `IF` condition is false.

```text
IF SYS_JOY_X > 500
    MOUSE_MOVE_REL 10 0
ELSE
    MOUSE_MOVE_REL -10 0
END
```

---

## `BREAK`

Immediately exits the innermost active loop.

```text
LOOP 100
    IF SYS_BATTERY < 10
        BREAK
    END
END
```

---

## `CONTINUE`

Skips the remainder of the current loop iteration and starts the next iteration.

```text
LOOP 10
    IF VAR_A == 0
        CONTINUE
    END
    TAP KEY_A
END
```

---

## `END`

Marks the end of a control-flow block.

Used with:

- `LOOP`
- `LOOP WHILE`
- `IF`
- `ELSE`

Example:

```text
IF VAR_A == 1
    TAP KEY_A
END
```

---

## `CALL [macro_id]`

Starts another macro by its unique macro ID.

```text
CALL 12
```

Useful for reusable functionality.

The VM must enforce a maximum call depth to prevent recursive macros from consuming unlimited resources.

---

## `RETURN`

Returns from a macro invoked using `CALL`.

```text
RETURN
```

If executed at the top level, it terminates the current macro instance.

---

# 3. Logic Operators

Logic operators are used inside conditions.

## `==`

Checks whether two values are equal.

```text
IF VAR_A == 10
    TAP KEY_A
END
```

---

## `!=`

Checks whether two values are different.

```text
IF SYS_LAYER != 0
    TAP KEY_B
END
```

---

## `<`

Checks whether the left value is smaller than the right value.

```text
IF SYS_BATTERY < 20
    LED_SET_COLOR #FF0000
END
```

---

## `>`

Checks whether the left value is larger than the right value.

```text
IF SYS_JOY_X > 500
    MOUSE_MOVE_REL 5 0
END
```

---

## `<=`

Checks whether the left value is smaller than or equal to the right value.

---

## `>=`

Checks whether the left value is larger than or equal to the right value.

---

## `AND`

Requires both conditions to be true.

```text
IF SYS_BATTERY < 20 AND SYS_JOY_X > 500
    LED_SET_COLOR #FF0000
END
```

---

## `OR`

Requires at least one condition to be true.

```text
IF SYS_LAYER == 1 OR SYS_LAYER == 2
    TAP KEY_SHIFT
END
```

---

## `NOT`

Inverts a boolean condition.

```text
IF NOT SYS_TOUCH_DOWN
    RELEASE KEY_SHIFT
END
```

---

# 4. Variables & Arithmetic

The VM provides general-purpose registers.

## `VAR_A`

General-purpose numeric register.

---

## `VAR_B`

General-purpose numeric register.

---

## `VAR_C`

General-purpose numeric register.

---

## `VAR_D`

General-purpose numeric register.

Variables can store temporary values used by macros.

---

## `SET [variable] [value]`

Assigns a value to a variable.

```text
SET VAR_A 100
```

After execution:

```text
VAR_A = 100
```

---

## `ADD [variable] [value]`

Adds a value to a variable.

```text
ADD VAR_A 10
```

Equivalent to:

```text
VAR_A = VAR_A + 10
```

---

## `SUB [variable] [value]`

Subtracts a value from a variable.

```text
SUB VAR_A 10
```

---

## `MUL [variable] [value]`

Multiplies a variable by a value.

```text
MUL VAR_A 2
```

---

## `DIV [variable] [value]`

Divides a variable by a value.

```text
DIV VAR_A 2
```

Division by zero must produce a controlled VM error rather than undefined behavior.

---

## `MOD [variable] [value]`

Calculates the remainder after division.

```text
MOD VAR_A 10
```

For example:

```text
VAR_A = 27
MOD VAR_A 10
```

results in:

```text
VAR_A = 7
```

---

# 5. Timing Keywords

## `WAIT [milliseconds]`

Suspends the current macro for the specified duration.

```text
TAP KEY_A
WAIT 100
TAP KEY_B
```

Important:

`WAIT` must be **non-blocking**.

It pauses only that macro instance. The controller continues processing:

- physical inputs
- other macros
- HID
- ESP-NOW
- LEDs
- system tasks

---

## `WAIT_UNTIL [condition]`

Suspends the current macro until a condition becomes true.

```text
WAIT_UNTIL SYS_JOY_X > 500
TAP KEY_A
```

This must also be implemented as a VM state rather than a blocking wait.

A timeout should be available for conditions that may never become true.

---

# 6. HID Keyboard Keywords

## `PRESS [key]`

Sends a keyboard key-down event.

```text
PRESS KEY_CTRL
```

The key remains logically pressed until a corresponding `RELEASE`.

---

## `RELEASE [key]`

Sends a keyboard key-up event.

```text
RELEASE KEY_CTRL
```

---

## `TAP [key]`

Presses and releases a key.

```text
TAP KEY_A
```

Conceptually:

```text
PRESS KEY_A
RELEASE KEY_A
```

The VM handles the timing/state management.

---

# 7. Mouse Keywords

## `MOUSE_CLICK [button]`

Clicks a mouse button.

Examples:

```text
MOUSE_CLICK LEFT
MOUSE_CLICK RIGHT
MOUSE_CLICK MIDDLE
```

---

## `MOUSE_MOVE_REL [x] [y]`

Moves the mouse relative to its current position.

```text
MOUSE_MOVE_REL 20 -5
```

Positive/negative direction follows the controller's configured coordinate convention.

---

## `MOUSE_MOVE_ABS [x] [y]`

Moves an absolute pointer to a normalized or configured coordinate.

```text
MOUSE_MOVE_ABS 0.5 0.5
```

The exact coordinate range must be defined by the HID implementation.

---

# 8. Joystick / Gamepad Keywords

## `JOYSTICK_SET [x] [y]`

Sets the virtual joystick position.

```text
JOYSTICK_SET 0 127
```

The values are normalized according to the HID/gamepad configuration.

The implementation must define:

- minimum
- center
- maximum
- signed/unsigned representation

---

# 9. Layer Keywords

Layers are temporary overlays on top of a mode/layout.

## `LAYER_ON [id]`

Activates a layer.

```text
LAYER_ON 1
```

Example use:

```text
ON_PRESS
    LAYER_ON 1
END
```

---

## `LAYER_OFF [id]`

Deactivates a layer.

```text
LAYER_OFF 1
```

---

## `LAYER_TOGGLE [id]`

Toggles a layer between active and inactive.

```text
LAYER_TOGGLE 1
```

---

# 10. Input Routing Keywords

Routing determines what an input behaves as.

## `ROUTE_SET [input] [output]`

Assigns an input to an output/action.

```text
ROUTE_SET JOYSTICK MOUSE
```

The exact input/output identifiers are defined by the routing system.

---

## `ROUTE_CLEAR [input]`

Removes a custom route from an input.

```text
ROUTE_CLEAR JOYSTICK
```

The input falls back to its normal mapping.

---

## `ROUTE_RESET`

Restores all routing to the active mode/layout defaults.

```text
ROUTE_RESET
```

---

# 11. LED Keywords

## `LED_SET_COLOR [#RRGGBB]`

Sets an RGB LED to a specific color.

```text
LED_SET_COLOR #FF0000
```

The example sets the LED to red.

---

## `LED_SET_HSV [H] [S] [V]`

Sets an RGB LED using HSV values.

```text
LED_SET_HSV 120 100 100
```

This makes dynamic color effects easier because hue, saturation, and brightness can be controlled independently.

---

## `LED_OFF`

Turns the selected LED off.

```text
LED_OFF
```

---

## `LED_BRIGHTNESS [value]`

Changes LED brightness without changing its current color.

```text
LED_BRIGHTNESS 50
```

The exact range should be defined as part of the LED API, preferably:

```text
0–100
```

---

# 12. ESP-NOW Keywords

## `TX_PAYLOAD [data]`

Transmits an arbitrary payload over ESP-NOW.

```text
TX_PAYLOAD "ARM"
```

Useful for custom receiver protocols.

---

## `TX_VALUE [value]`

Transmits a numeric value.

```text
TX_VALUE VAR_A
```

Useful for sending:

- joystick values
- encoder values
- macro parameters
- telemetry

---

## `TX_STRING [string]`

Transmits a text/string payload.

```text
TX_STRING "MODE_CAD"
```

The receiver must implement the corresponding protocol.

---

# 13. System Variables

System variables expose live controller state to macros.

## `SYS_JOY_X`

Current processed joystick X-axis value.

Useful for:

- mouse control
- conditional macros
- analog thresholds
- custom curves

---

## `SYS_JOY_Y`

Current processed joystick Y-axis value.

---

## `SYS_TOUCH_X`

Current touchpad X coordinate.

The coordinate should use the normalized touchpad coordinate system.

---

## `SYS_TOUCH_Y`

Current touchpad Y coordinate.

---

## `SYS_TOUCH_DOWN`

Boolean indicating whether the touchpad is currently being touched.

Example:

```text
IF SYS_TOUCH_DOWN
    ...
END
```

---

## `SYS_ENC_1_POS`

Current logical position of Encoder #1.

---

## `SYS_ENC_2_POS`

Current logical position of Encoder #2.

---

## `SYS_BATTERY`

Current estimated battery percentage.

This should be the filtered/estimated battery level rather than raw ADC data.

---

## `SYS_LAYER`

Current active layer or resolved layer state.

---

## `SYS_MODE`

Current mode identifier.

---

## `SYS_LAYOUT`

Current layout identifier.

---

# 14. Input Configuration

## Buttons

Buttons support:

- press
- release
- hold
- long press
- repeat
- macro trigger
- HID action
- layer action
- route changes

Each button can independently define `ON_PRESS` and `ON_RELEASE`.

---

# 15. Rotary Encoders

Encoders support:

- clockwise movement
- counter-clockwise movement
- absolute logical position
- push button
- acceleration
- configurable step size
- macro selection
- HID actions

Encoder #2 can operate as a macro selector.

Example:

```text
Clockwise   → next macro
Counterclockwise → previous macro
Press       → execute selected macro
```

---

# 16. Joystick

The joystick can operate in several modes.

## `MOUSE_XY`

Maps joystick movement to relative mouse movement.

Supports:

- deadzone
- sensitivity
- response curve
- acceleration
- axis inversion

---

## `DPAD`

Converts analog joystick position into digital directional inputs.

Example:

```text
up
down
left
right
```

A configurable deadzone prevents accidental activation near center.

---

## `RAW_ANALOG`

Exposes calibrated analog values directly to the macro/routing system.

Useful for custom applications.

---

# 17. Trackpad Modes

## `RELATIVE_MOUSE`

Converts touch movement into relative mouse movement.

---

## `ABSOLUTE_THUMBSTICK`

Maps touchpad coordinates to an absolute joystick-style coordinate.

---

## `SCROLL_WHEEL`

Converts vertical/horizontal touch movement into scrolling.

---

## `IPOD_SCRUBBER`

Uses the touch position around a configurable center point.

Polar coordinates are used to determine:

- direction
- angular movement
- radial movement

Useful for media scrubbing or circular controls.

---

## `MACRO_ZONE`

Turns a physical region of the touchpad into a virtual button or macro trigger.

Example:

```text
left edge → hold SHIFT
top-right → execute MACRO_12
center → mouse
```

---

# 18. Modes

A mode is a complete controller profile.

A mode can define:

- input mappings
- macros
- LED behavior
- encoder behavior
- joystick behavior
- trackpad behavior
- layers
- ESP-NOW behavior
- Encoder #2 macro table

Example modes:

```text
3D_MODELING
DRONE
GAMING
CAD
CUSTOM
```

There is no requirement for a fixed number of modes.

---

# 19. Layouts

The controller has two base layouts:

```text
LAYOUT_A
LAYOUT_B
```

The physical A/B switch selects the active layout.

Layout B can inherit values from Layout A when no override is configured.

This allows a layout to change only a few controls instead of duplicating the entire configuration.

---

# 20. Layers

Layers provide temporary control overlays.

Example:

```text
BASE
SHIFT
PRECISION
MACRO
```

A shift button can activate a layer while held:

```text
ON_PRESS
    LAYER_ON 1
END

ON_RELEASE
    LAYER_OFF 1
END
```

This allows one physical control to have different functions depending on context.

---

# 21. Encoder #2 Macro Library

Macros are stored in a global library.

Modes can assign a list of macro IDs to Encoder #2.

Example:

```text
CAD Mode

Encoder #2:
    0 → Zoom
    1 → Pan
    2 → Orbit
    3 → Measure
    4 → Screenshot
```

Turning the encoder selects an entry.

Pressing the encoder executes the selected macro.

The selected index belongs to the active mode's table, not the global macro-library order.

---

# 22. Trackpad Slicer

The configuration UI provides an interactive canvas for dividing the trackpad into zones.

Each zone has:

```text
position
size
priority
mode
gesture
press action
release action
hold action
macro
```

Coordinates should be stored in normalized form:

```text
X = 0.0–1.0
Y = 0.0–1.0
```

rather than browser pixel coordinates.

If zones overlap, explicit priority determines which zone receives the event.

---

# 23. Macro Suppression

Macros can suppress the normal native action of an input.

Conceptually:

```text
physical press
    ↓
macro trigger
    ↓
macro requests SUPPRESS
    ↓
normal HID action is suppressed
```

This is useful when an input normally produces a HID action but a macro needs to replace that behavior.

Suppression must be cleared when the corresponding input is released or the input state is reset.

---

# 24. Configuration Web UI

The controller hosts its own configuration interface.

The UI provides:

- mode management
- layout management
- layer management
- input configuration
- macro creation/editing
- macro assignment
- trackpad zone editing
- joystick configuration
- encoder configuration
- LED configuration
- ESP-NOW configuration
- live telemetry
- profile management
- OTA firmware updates

---

# 25. Macro Editor

The macro editor supports:

### Visual mode

Drag-and-drop blocks representing macro commands.

### Text mode

Users write MacroScript directly.

The browser converts the script into bytecode:

```text
MacroScript
    ↓
Parser
    ↓
Validation
    ↓
Bytecode compiler
    ↓
WebSocket
    ↓
ESP32
```

The ESP32 validates the received bytecode again before executing it.

---

# 26. Macro Execution Rules

Macros are stateful and non-blocking.

Multiple macros may exist simultaneously, subject to a configured maximum.

A macro waiting on `WAIT` does not stop other macros from executing.

Example:

```text
Macro A
    TAP KEY_A
    WAIT 1000
    TAP KEY_B
END
```

While Macro A is waiting, the controller remains fully responsive.

---

# 27. VM Safety Limits

The VM must enforce hard limits for:

```text
maximum macro size
maximum instructions per tick
maximum call depth
maximum loop depth
maximum active macros
maximum runtime
maximum variable range
```

These prevent accidental or malicious macros from monopolizing the controller.

---

# 28. HID State Recovery

The firmware tracks which HID keys/buttons it believes are currently pressed.

If a macro crashes, is canceled, or the controller resets, the firmware can release outstanding HID states.

After reset:

```text
keyboard = released
mouse buttons = released
joystick = neutral
VM = stopped
temporary layers = cleared
suppression = cleared
```

This prevents "stuck key" behavior.

---

# 29. Overall Software Pipeline

```text
Physical Input
      │
      ▼
Input Processing
      │
      ▼
Mode / Layout / Layer Resolution
      │
      ▼
Input Router
      │
      ├───────────────┐
      ▼               ▼
Native HID         Macro VM
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
            HID    ESP-NOW   LEDs/Buzzer
```

The key design principle is:

> **Inputs are normalized first, routing decides what they mean, and the macro VM provides programmable behavior without blocking the real-time controller.**
