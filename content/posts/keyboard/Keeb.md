# Keeb

## 一些 Essentials
对于键盘来说，从用户的角度，是由按键组成的（编码器/虚拟按键等，暂不考虑）。

一个一个的按键，组成了键盘的布局，用户根据布局操作每一个按键。
因此，layout是最上层的概念。对于软件来说，layout包含如下特性：
1. layout共有多少行、列
2. layout的每一个位置和PCB中的矩阵的对应关系，哪些位置有按键，哪些没有。
3. layout存的是键位的设置，而每个键位的具体功能，则存在和layout对应的keymap中。

在Keymap中，存储的是每个按键的按下后的具体功能 Action 。一个keymap可以有多个layer，keymap和layer的关系，可以参考 https://docs.qmk.fm/#/keymap 。
   
在实际用户使用的时候，首先是要获取用户触发的键位 position ，然后判断用户的 TriggerType（触发方式）。
对于键盘来说，每一个按键可以有如下的触发方式：
1. 按下/抬起：最普通的触发方式
2. tap：把快速按下+抬起组合起来，看做是一个动作。只有完成了整个的tap动作，才会触发。tap一般都有一个时间上限，在时间上限内完成 按下+抬起，可以识别为tap
3. tapN：连续N次tap
4. hold：长按，有一个最低时间。
5. NO：没有动作，默认的

然后，根据对应 keymap + position 里面存储的 Action，再加上 TriggerType，就能得到键盘最终执行的动作。

即 Position + TriggerType + Action = What's executed by keyboard(aka KeyboardAction).

然后，键盘 dispatch 所有的 KeyboardAction，去执行相关操作。

## Definitions

- keycode：键码
  - 键码应该有 2 套：
    - 系统支持的键码
    - 内部的键码，包含系统支持键码 + 内部功能（比如切层、RGB、音频控制等等）
- layout：布局
  - 需要支持KLE
  - 要点：键位、PCB矩阵位置、形状 & 角度
  - layout需要和具体的键位position绑定。然后，每个键位有X种触发方式，每一种触发方式有Y种功能。
- matrix：PCB矩阵，和layout存在有限的对应
- layer：层
  QMK中，一共支持32层，可以同时激活多个层，多个层同时激活的时候，会从高往低逐个扫描，如果是KC_TRANS，就进到低层，直到在某层有一个有效的按键。支持如下切层功能：
    1. DF(layer): 设置默认层，默认层默认一直是激活状态。。默认层一般都是第0层，也就是最下面的一层
    2. MO(layer): 临时激活某一层，比如MO(1)就是临时激活第1层，松开这个按键的时候失效
    3. LM(layer, mod): 也是临时激活某层，区别是LM自带某个modifier（不用手再去按了）
    4. LT(layer, kc): 双功能，长按是激活某一层，单击是触发对应的keycode
    5. OSL(layer): 按一下临时激活某层，激活状态在下次按键失效。和MO(layer)的区别就是，MO(layer)需要保持按下激活某一层，而OSL(layer)是一次性的，只对下一个按键有效
    6. TG(layer): 切换某层的激活状态。上面的都是临时激活层，而TG是永久切换层的激活状态(开 -> 关/关 -> 开)
    7. TO(layer): 激活某一层，关闭其他所有层。这个在按下的那一刻起就生效了。
    8. TT(layer): 按下临时激活某层（和MO一样），但是增加了一个功能是，如果连续单击TT，则可以把这一层永久激活或关闭（相当于TG）。换句话说，TT是双功能，等于MO(按下) + TG(多击)。多击次数可以通过 `TAPPING_TOGGLE`设置，如在QMK中，设置`#define TAPPING_TOGGLE 2`，就是把TT设置成双击激活。

- macro：宏
- action：键盘的动作
  除了基础的发送键码之外，键盘的按键还有其他很多功能，如：
  1. 层操作：临时切层/激活&失活某层/层+modifier/层的tap触发
  2. modifier 
  3. tap
  4. 媒体控制（播放、暂停、音量）
  5. 系统控制（关机）
  这些功能并不能被键盘的keycode覆盖，统称为 keyboard action
  
  另外，再做一层抽象：人操作键盘有几种方式：
  1. 按下/抬起：最正常的方式，按下和抬起分别算
  2. tap：也就是按下 + 抬起触发
  3. hold：长按
  4. 连续tap：qmk中的tap dance
  5. 组合键：同时按下多个按键，可以是 modifier + key，也可以是key + key

  在上面的这些，可以理解成是human action


  

- debounce
- 协议层：支持via、vial
- 驱动层：trait for GPIO、LED、RGB、Encoder等等
- tools：快捷生成、快捷接入等
- cli：命令行工具

## Keycode

### System keycode

首先是系统接受的键码，这些键码是国际标准定义好的，可以参见QMK的文档。

tap key: 按一下就触发一次

## Layer

QMK一共支持32层，层的操作上面有写。在具体实现上，

  1. 由于每层都是独立的，因此层的开启和关闭，主要使用bitwise operation来实现。
  2. 层 + modifier需要单独实现
  3. 层的 tap 操作需要单独实现


## About Rust

### 变量

rust中变量默认不可变，要可变需要加`mut`

rust中，还可以用 `cosnt` 去声明一个常量。声明const的时候，必须指定类型。const一般用大写字母表示。

### 基础类型

#### 数字

对于数字和基础的变量，rust支持多种进制的表示。还可以添加`_`作为虚拟分隔符（主要是为了可读性）。

| Number literals  | Example       |
| ---------------- | ------------- |
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |
| Byte (`u8` only) | `b'A'`        |

#### 浮点数、bool

f32 & f64 & bool，这些通用的，基本都差不多

#### Tuple

Rust里面也有tuple，用 () 括起来

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
```

还能解组

```rust
let (x, y, z) = tup;
```

Tuple取值，使用 `.`：`let x = tup.0`

#### array

Array 是一个序列，和tuple的区别是，Array中的**数据类型必须一致**。

Array还有一个特点：它被分配在stack上，**其长度是固定的**

```rust
let a: [i32; 5] = [3; 5];
```

Array的取值，和tuple也不同：`let x = a[0]`

直接取的话，数组越界会直接panic，需要注意

