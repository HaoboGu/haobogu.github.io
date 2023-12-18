---
title: "Learn QMK - 2"
author: "Haobo Gu"
tags: [keyboard, qmk]
date: 2022-08-14T15:38:55+08:00
summary: Learn about customization of QMK
---

# QMK - 2

QMK's source code can be found at Github: https://github.com/qmk/qmk_firmware.

## Learn QMK's source code

### Basic project structure

There are three logic levels in QMK project:

```
Core(_quantum)
    -- Keyboard/Revision(_kb)
        -- Keymap(_user)
```

 In QMK, many custom functions have a `_kb` or `_user` suffix. By convention, when you customize your keyboard or a revision of your keyboard, using `_kb` functions. And when you customize your keymap, use `_user` functions.

Remember to call `_user` function at the beginning of your `_kb` functions. Otherwise, those `_user` functions won't be execute any more.

### Program Entry

Like other C programs, QMK's entry is a `main()` function. QMK's main function is at `quantum/main.c`, which is the entrance of all the QMK firmware. 

QMK's `main()` function is quite simple: setup platform/protocol/keyboard and then run the infinite main loop. The main loop will call `protocol_task()`, then the `keyboard_task()` in  [`quantum/keyboard.c`](https://github.com/qmk/qmk_firmware/blob/0.15.13/quantum/keyboard.c#L377) is called. `keyboard_task()` is where the keyboard specific functionality is dispatched, such as matrix scanning, mouse handling and controlling keyboard status LEDs.

### Matrix Scanning

Matrix scanning is the core of a keyboard firmware. QMK provides built in scanning algorithm, you just need to define your matrix layout. 

To declare your own key matrix, a C macro is used. For example, to define a 2*2 matrix, you can use the following code

```c
#define LAYOUT( \
  k00, k01, \
  k10, k11 \
) { \
  {k00, k01}, \
  {k10, k11} \
}
```

Note that you may not have a key at every position in the matrix, you can use keycode `KC_NO` in the second part of the macro. A typical numpad layout can be defined using the following code:

```c
#define LAYOUT( \
    k00, k01, k02, k03, \
    k10, k11, k12, k13, \
    k20, k21, k22, \
    k30, k31, k32, k33, \
    k40,      k42 \
) { \
    { k00, k01,   k02, k03   }, \
    { k10, k11,   k12, k13   }, \
    { k20, k21,   k22, KC_NO }, \
    { k30, k31,   k32, k33   }, \
    { k40, KC_NO, k42, KC_NO } \
}
```

In keymap, you can use this macro to map keycodes of actual physical keys to matrix keys:

```c
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [0] = LAYOUT(
        KC_NUM,  KC_PSLS, KC_PAST, KC_PMNS,
        KC_P7,   KC_P8,   KC_P9,   KC_PPLS,
        KC_P4,   KC_P5,   KC_P6,
        KC_P1,   KC_P2,   KC_P3,   KC_PENT,
        KC_P0,            KC_PDOT
    )
}
```

You can see the keymap has 3 dimensions. The first dimension is actual **layer**. Each layer has `MATRIX_ROWS` * `MATRIX_COLS` keys. In the given example, we defined only one layer. 

### Detect key strokes

At each matrix scanning loop, the matrix scanning function returns the current state of the matrix. QMK stores the result of last matrix scan, and compares with the current scanning result to determine which key is pressed or released. The the key code is dispatched to `process_record()` function.

### Process Record

Function `process_record()` is not complex, it contains a chain of events(c functions). Many of then depends on rules defined in `rules.mk`. The full events list is [here](https://docs.qmk.fm/#/zh-cn/understanding_qmk?id=process-record). If any of them returns `false`, the following functions won't be executed.

## Customize keymap

You can create your own key code with QMK. To create your key code, you need to define an enum in `keymap.c` first:

```c
enum my_keycodes {
  FOO = SAFE_RANGE,
  BAR
};
```

QMK provides a macro `SAFE_RANGE` which ensure that you got a unique and correct key code.

### Program a key

When you want to overwrite your key's function or define the functionality of your new keycode, you can use `process_record_kb()` or `process_record_user()`. Remember this `process_record()` function? Your customized `process_record_kb/user()` function is similar: return false if you want to overwrite this key or attach new functionality to the key, otherwise just return true. Here is an example:

```c
// Example of process_record_
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
  switch (keycode) {
    case FOO:
      // Set your new key
      if (record->event.pressed) {
        // 按下时做些什么
      } else {
        // 抬起时做些什么
      }
      // 覆盖已有功能
      return false; // 跳过此键的所有进一步处理
    case KC_ENTER:
      // 给回车增加新的功能
      // 当按下回车时播放音符
      if (record->event.pressed) {
        PLAY_SONG(tone_qwerty);
      }
      // 返回true则不覆盖其原来的功能，返回false则为覆盖
      return true; // 让QMK响应回车按下/抬起事件
    default:
      // 其他键位都保持不变（返回true）
      return true; // 正常响应其他键码
  }
}
```

The definition of input param `record`：

```c
keyrecord_t record {
  keyevent_t event {
    keypos_t key {
      uint8_t col
      uint8_t row
    }
    bool     pressed
    uint16_t time
  }
}
```

`record` has the input key's col/row, whether it's pressed and the press time.

## Keyboard Initialization

You can also customize the initialization process of the keyboard. There are 3 functions that you can overwrite:

- `keyboard_pre_init_*`: happens at the early start of the firmware's setup process, can be used to initialize your hardware
- `matrix_init_*`: happens midway of the firmware's setup process. At this moment, hardware is initialized, but features may not be yet
- `keyboard_post_init_*`: happens at the end of the firmware's setup process. Hardware and most features are ready, you should put most of your customization code here

### Keyboard Pre Initialization

`keyboard_pre_init_*` is used to initialize your own extra hardwares. Note that this process starts very early -- even earlier than the USB starts. The following is an example to set up LEDs using `keyboard_pre_init_*`:

```c
void keyboard_pre_init_user(void) {
  // Call the keyboard pre init code.

  // Set our LED pins as output
  setPinOutput(B0);
  setPinOutput(B1);
  setPinOutput(B2);
  setPinOutput(B3);
  setPinOutput(B4);
}
```

### Matrix Initialization Code

`matrix_init_*` is called when the matrix is initialized, you can overwrite the low-level matrix configuration here. If you want to change the default pin initialization method, you can use the following method:

- `void matrix_init_pins(void)`: GPIO pin initialization. By default it will initialize pins set in `MATRIX_ROW_PINS` and `MATRIX_COL_PINS`, and the setup method is defined using `ROW2COL`, `COL2ROW` or `DIRECT_PINS`.

### Keyboard Post Initialization Code

You most customization code should be wrote here. At this moment, most hardware and features are initialized, you can take changes to certain features as you want.

The following is an example about setting up RGB lights:

```c
void keyboard_post_init_user(void) {
  // Call the post init code.
  rgblight_enable_noeeprom(); // enables Rgb, without saving settings
  rgblight_sethsv_noeeprom(180, 255, 255); // sets the color to teal/cyan without saving
  rgblight_mode_noeeprom(RGBLIGHT_MODE_BREATHING + 3); // sets mode to Fast breathing without saving
}
```

## QMK configuration files

We've already knew that there are some configuration files in QMK when you're going to create your own keyboard, such as `info.json`, `rules.mk`. In this section, we'll learn how to customize those files.

NOTE: there are many configs that you can set at multiple files. In this case, when you try to compile the firmware, qmk would complain and tell you which config will be used:

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220815221008303.png" alt="image-20220815221008303" style="zoom:33%;" />

Prune your configurations accordingly.

### info.json

You can config the hardware and enabled features in `info.json`. The full reference of `info.json` can be found here: https://docs.qmk.fm/#/reference_info_json.

#### Hardware configuration

At the top of `info.json`, there are some configs about the manufacturer and hardware that you can customize:

```json
{
    "keyboard_name": "your_keyboard_name",
    "manufacturer": "You",
    "maintainer": "You",
    "usb": {
        "vid": "0xFEED",
        "pid": "0x0000",
        "device_version": "1.0.0"
    }
}
```

in which, the `manufacturer` and `keyboard_name` will be displayed in the list of USB devices on Windows and MacOS. You can also choose your USB's vid and pid. 

#### Matrix configuration

You can also define the GPIO pins which are used for matrix scanning in `info.json`:

```json
{
  "matrix_pins": {
    "cols": ["C1", "C2", "C3", "C4"],
    "rows": ["D1", "D2", "D3", "D4"]
  },
}
```

Then declare your diode direction in your PCB:

```json
"diode_direction": "ROW2COL"
```

#### Layout configuration

Next you can config the layout of your keyboard:

```json
    "layouts": {
        "LAYOUT_fancer": {
            "layout": [
                { "matrix": [0, 0], "x": 0, "y": 0 },
                { "matrix": [0, 1], "x": 1, "y": 0 },
                { "matrix": [1, 0], "x": 0, "y": 1 },
                { "matrix": [1, 1], "x": 1, "y": 1 }
            ]
        }
    }
```

For more available configurations, see: https://docs.qmk.fm/#/reference_info_json

### config.h

`config.h` is the basic header of the firmware. All configurations here are persist over the whole project. There are lots of configurations available, see this [document](https://docs.qmk.fm/#/config_options?id=the-configh-file).

I list some most commonly used configs in `config.h` here:

```c
// vendor id
#define VENDOR_ID 0x1234
// product id
#define PRODUCT_ID 0x1234
// device version(often used for revisions)
#define DEVICE_VER 0
// manufacturer
#define MANUFACTURER MY_LAB
// keyboard name
#define PRODUCT fancer
// number of rows/cols
#define MATRIX_ROWS 2
#define MATRIX_COLS 2
// MCU pins of rows, from top to down(might be overwritten by user, see https://docs.qmk.fm/#/custom_quantum_functions?id=low-level-matrix-overrides)
#define MATRIX_ROW_PINS { A1, A3 }
// MCU pins of cols, from left to right(might be overwritten as well)
#define MATRIX_COL_PINS { A2, A4 }
// IO delay in ms between changing matrix pin and reading values
#define MATRIX_IO_DELAY 30
// the direction of diode, COL2ROW means the black mark on your diode is facing to the rows
#define DIODE_DIRECTION COL2ROW
// debounce threshold in ms
#define DEBOUNCE 5
// layout of the keyboard
#define LAYOUT( \
    k00, k01,  \
    k10, k11,  \
) { \
    { k00, k01, }, \
    { k10, k11, }, \
}
```

### rules.mk

`rules.mk` defines some built options when compiling the firmware. It's actually a **makefile** which can be recognized with tools like `cmake`. In `rules.mk` you can set building configurations, MCU options and enable certain features.

#### Build Options

- `FIRMWARE_FORMAT`: defines the compiled firmware's format, like `.hex` or `.bin`
- `SRC`: added sources files for compilation/linking
- `LAYOUTS`: a list of layouts that this keyboard supports. See Matrix Scanning section.
- `LTO_ENABLE`: add it to reduce the size of your firmware

#### Feature Options

You can also enable/disable many features such as MAGIC Actions, AUDIO, RGBLIGHT, etc. in `rules.mk`. The full list can be found here: https://docs.qmk.fm/#/config_options?id=feature-options.

Here is an example of `rules.mk` of owlab suit80:

```makefile
# MCU name
MCU = atmega32u4

# Bootloader selection
BOOTLOADER = atmel-dfu

# Build Options
#   change yes to no to disable
#
BOOTMAGIC_ENABLE = yes      # Enable Bootmagic Lite
MOUSEKEY_ENABLE = yes       # Mouse keys
EXTRAKEY_ENABLE = yes       # Audio control and System control
CONSOLE_ENABLE = no         # Console for debug
COMMAND_ENABLE = no         # Commands for debug and configuration
NKRO_ENABLE = yes           # Enable N-Key Rollover
BACKLIGHT_ENABLE = no       # Enable keyboard backlight functionality
RGBLIGHT_ENABLE = no        # Enable keyboard RGB underglow
AUDIO_ENABLE = no           # Audio output
```





