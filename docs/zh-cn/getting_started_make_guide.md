# 深入介绍 `make`

<!---
  original document: 0.16.0:docs/getting_started_make_guide.md
  git diff 0.16.0 HEAD -- docs/getting_started_make_guide.md | cat
-->

`make` 指令的完整语法形式为 `<keyboard_folder>:<keymap>:<target>`，其中：

* `<keyboard_folder>` 为键盘所处目录的名字，比如 `planck`
  * 使用 `all` 可以编译所有的键盘
  * 使用更详尽的路径来编译指定的修订版，比如 `planck/rev4` 或 `planck/rev3`
  * 若键盘没有对应的目录，可以不提供此参数
  * 若构建的是默认的目录，可以不提供此参数
* `<keymap>` 为键映射所处目录的名字，比如 `algernon`
  * 使用 `all` 可以编译所有的键映射
* 我们会在接下来的内容详细解释 `<target>` 部分。

`<target>` 的用法有：
* 若不指定 target，则与下一条的 `all` 效果一致
* 使用 `all` 可以编译给定的键盘/修订版/键映射的所有可能的组合形式。例如，`make planck/rev4:default` 会生成一个 .hex 文件，而 `make planck/rev4:all` 将对 planck 的每个键映射生成一个 .hex 文件。
* `flash`, `dfu`, `teensy`, `avrdude`, `dfu-util`, 或 `bootloadhid` 会编译并上传固件到键盘上。若编译失败了，则不会上传。使用的编程器取决于是什么键盘，大部分键盘将使用 `dfu`，而 ChibiOS 键盘应使用 `dfu-util`，`teensy` 则是用于标准的 Teensys 键盘。各键盘应该使用什么指令，可以参阅其 readme 文件。
  在[刷写固件](zh-cn/flashing.md)指引中有可用的 bootloader 的更详尽的信息。
  * **注意**：部分操作系统中执行这些指令需要对应的访问权限，因此你可能需要先配置 [`udev 规则`](faq_build.md#linux-udev-rules) 才可以脱离 root 权限访问设备，或者使用 root 权限来执行命令（`sudo make planck/rev4:default:flash`）。
* `clean`，可以清空构建输出目录以确保所有文件都是重新构建的。若你遇到了难以解释的诡异问题，请在常规构建前执行此命令。
* `distclean` 将清除 .hex 及 .bin 文件。

以下构建目标主要用于辅助开发者：

* `show_path` 将展示出源文件及 object 文件的路径。
* `dump_vars` 输出所有的 makefile 变量。
* `objs-size` 输出每个 object 文件的大小。
* `show_build_options` 输出 'rules.mk' 内设置的所有配置项。
* `check-md5` 输出最终生成文件的 md5 校验值

在 make 命令行的末尾，构建目标的后面，可以追加如下的额外参数

* `make COLOR=false` - 关闭输出着色
* `make SILENT=true` - 关闭常规输出仅保留错误/警告信息
* `make VERBOSE=true` - 输出所有 gcc 相关信息（没什么营养，除非你需要调试）
* `make VERBOSE_LD_CMD=yes` - 执行 ld 命令时附加 -v 选项
* `make VERBOSE_AS_CMD=yes` - 执行 as 命令时附加 -v 选项
* `make VERBOSE_C_CMD=<c_source_file>` - 构建给定的 C 源文件时，附加 -v 选项
* `make DUMP_C_MACROS=<c_source_file>` - 构建给定的 C 源文件时，输出预处理器宏信息
* `make DUMP_C_MACROS=<c_source_file> > <logfile>` - 构建给定的 C 源文件时，输出预处理器宏信息到 `<logfile>` 文件中
* `make VERBOSE_C_INCLUDE=<c_source_file>` - 构建给定的 C 源文件时，输出所有被包含的文件的文件名
* `make VERBOSE_C_INCLUDE=<c_source_file> 2> <logfile>` - 构建给定的 C 源文件时，输出所有被包含的文件的文件名到 `<logfile>` 文件中

make 命令自身也有一些额外的参数，可以输入 `make --help` 来了解。最有用的可能是 `-jx`，可以让其在更多的 CPU 上执行编译，`x` 代表着可用的 CPU 数量。设置此参数可以大大减少编译用时，特别是在构建大量的键盘/键映射是。我通常会将其设置为比我的 CPU 总数少一个，这样在编译时我还能留有一部分 CPU 资源去做其他的事情。但留意并不是所有的操作系统和 make 版本都支持此参数。

以下列举一些指令示例

* `make all:all` 构建一切（所有键盘目录，及所有的键映射）。在 `root` 目录下执行 `make` 效果一致。
* `make ergodox_infinity:algernon:clean` 将清除 Ergodox Infinity 键盘的所有构建产出文件。
* `make planck/rev4:default:flash COLOR=false` 构建键映射并上传到键盘中，构建过程中的输出信息不着色。

## `rules.mk` 配置项

以下变量设置为 `no` 即代表禁用，设置为 `yes` 代表启用。

`BOOTMAGIC_ENABLE`

This allows you to hold a key (usually Escape by default) to reset the EEPROM settings that persist over power loss and ready your keyboard to accept new firmware.

`MOUSEKEY_ENABLE`

This gives you control over cursor movements and clicks via keycodes/custom functions.

`EXTRAKEY_ENABLE`

This allows you to use the system and audio control key codes.

`CONSOLE_ENABLE`

This allows you to print messages that can be read using [`hid_listen`](https://www.pjrc.com/teensy/hid_listen.html).

By default, all debug (*dprint*) print (*print*, *xprintf*), and user print (*uprint*) messages will be enabled. This will eat up a significant portion of the flash and may make the keyboard .hex file too big to program.

To disable debug messages (*dprint*) and reduce the .hex file size, include `#define NO_DEBUG` in your `config.h` file.

To disable print messages (*print*, *xprintf*) and user print messages (*uprint*) and reduce the .hex file size, include `#define NO_PRINT` in your `config.h` file.

To disable print messages (*print*, *xprintf*) and **KEEP** user print messages (*uprint*), include `#define USER_PRINT` in your `config.h` file (do not also include `#define NO_PRINT` in this case).

To see the text, open `hid_listen` and enjoy looking at your printed messages.

**NOTE:** Do not include *uprint* messages in anything other than your keymap code. It must not be used within the QMK system framework. Otherwise, you will bloat other people's .hex files.

`COMMAND_ENABLE`

This enables magic commands, typically fired with the default magic key combo `LSHIFT+RSHIFT+KEY`. Magic commands include turning on debugging messages (`MAGIC+D`) or temporarily toggling NKRO (`MAGIC+N`).

`SLEEP_LED_ENABLE`

Enables your LED to breath while your computer is sleeping. Timer1 is being used here. This feature is largely unused and untested, and needs updating/abstracting.

`NKRO_ENABLE`

This allows the keyboard to tell the host OS that up to 248 keys are held down at once (default without NKRO is 6). NKRO is off by default, even if `NKRO_ENABLE` is set. NKRO can be forced by adding `#define FORCE_NKRO` to your config.h or by binding `MAGIC_TOGGLE_NKRO` to a key and then hitting the key.

`BACKLIGHT_ENABLE`

This enables the in-switch LED backlighting. You can specify the backlight pin by putting this in your `config.h`:

    #define BACKLIGHT_PIN B7

`MIDI_ENABLE`

This enables MIDI sending and receiving with your keyboard. To enter MIDI send mode, you can use the keycode `MI_ON`, and `MI_OFF` to turn it off. This is a largely untested feature, but more information can be found in the `quantum/quantum.c` file.

`UNICODE_ENABLE`

This allows you to send Unicode characters using `UC(<code point>)` in your keymap. Code points up to `0x7FFF` are supported. This covers characters for most modern languages, as well as symbols, but it doesn't cover emoji.

`UNICODEMAP_ENABLE`

This allows you to send Unicode characters using `X(<map index>)` in your keymap. You will need to maintain a mapping table in your keymap file. All possible code points (up to `0x10FFFF`) are supported.

`UCIS_ENABLE`

This allows you to send Unicode characters by inputting a mnemonic corresponding to the character you want to send. You will need to maintain a mapping table in your keymap file. All possible code points (up to `0x10FFFF`) are supported.

For further details, as well as limitations, see the [Unicode page](feature_unicode.md).

`AUDIO_ENABLE`

This allows you output audio on the C6 pin (needs abstracting). See the [audio page](feature_audio.md) for more information.

`VARIABLE_TRACE`

Use this to debug changes to variable values, see the [tracing variables](unit_testing.md#tracing-variables) section of the Unit Testing page for more information.

`KEY_LOCK_ENABLE`

This enables [key lock](feature_key_lock.md).

`SPLIT_KEYBOARD`

This enables split keyboard support (dual MCU like the let's split and bakingpy's boards) and includes all necessary files located at quantum/split_common

`SPLIT_TRANSPORT`

As there is no standard split communication driver for ARM-based split keyboards yet, `SPLIT_TRANSPORT = custom` must be used for these. It will prevent the standard split keyboard communication code (which is AVR-specific) from being included, allowing a custom implementation to be used.

`CUSTOM_MATRIX`

Lets you replace the default matrix scanning routine with your own code. For further details, see the [Custom Matrix page](custom_matrix.md).

`DEBOUNCE_TYPE`

Lets you replace the default key debouncing routine with an alternative one. If `custom` you will need to provide your own implementation.

`DEFERRED_EXEC_ENABLE`

Enables deferred executor support -- timed delays before callbacks are invoked. See [deferred execution](custom_quantum_functions.md#deferred-execution) for more information.

## Customizing Makefile Options on a Per-Keymap Basis

If your keymap directory has a file called `rules.mk` any options you set in that file will take precedence over other `rules.mk` options for your particular keyboard.

So let's say your keyboard's `rules.mk` has `BACKLIGHT_ENABLE = yes`. You want your particular keyboard to not have the backlight, so you make a file called `rules.mk` and specify `BACKLIGHT_ENABLE = no`.
