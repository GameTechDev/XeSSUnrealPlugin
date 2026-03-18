# Intel® XeSS Plugin for Unreal Engine

## Introduction

Intel® XeSS Plugin for Unreal Engine integrates Intel® [XeSS 3 technologies](https://www.intel.com/content/www/us/en/developer/topic-technology/gamedev/xess.html) into Unreal Engine (UE). XeSS Super Resolution (XeSS-SR), XeSS Frame Generation (XeSS-FG) including Multi-Frame Generation and Xe Low Latency (XeLL) are parts of XeSS 3.

For more information, please visit: [https://github.com/intel/xess](https://github.com/intel/xess)

If you encounter any integration issues, feel free to use the [Intel® XeSS Inspector](https://intel.com/xess-inspector). This tool is specifically designed to simplify the validation and debugging of XeSS integration within applications.

## Downloads

Please open the [Releases](https://github.com/GameTechDev/XeSSUnrealPlugin/releases) page to download compiled plugin packages.

## Installing the plugin

1\. If you use the source code version of UE, you need to build the plugin locally, otherwise skip this step.

* Open the terminal and go to `<ue_root_dir>\Engine\Build\BatchFiles`

* Run the command line:

```text
RunUAT.bat BuildPlugin -Plugin="<path_to_downloaded_plugin>\XeSS.uplugin" -Package="<path_to_destination_plugin>" -TargetPlatforms=Win64 -VS2019
```

> The last parameter is the version of Visual Studio used to build, required by UE 4 only.

2\. Copy (pre-)built files to:

* For UE 4: `<ue_root_dir>\Engine\Plugins\Runtime\Intel\XeSS`

* For UE 5: `<ue_root_dir>\Engine\Plugins\Marketplace\XeSS`
  > **Notes:** Please maintain this exact folder structure when installing the plugin. Deviating from this path may cause failures when packaging your game from the Editor. If folder `Marketplace` doesn't exist, please create it manually.

<div style="page-break-after: always;"></div>

## Enabling the plugin in the Editor

To enable the XeSS plugin for a project, go to the plugin selection.

![Alt text](Images/Plugin_settings.png?raw=true "UE 4 project settings")

Use the search box to find the XeSS plugin and make sure it is enabled.

![Alt text](Images/Enabled_xess_plugin.png?raw=true "Enable XeSS plugin")

<div style="page-break-after: always;"></div>

## Project settings

Once this plugin is enabled, its default behavior can be modified in the Project Settings menu.

It is possible to disable/enable it in the Editor viewports.

![Alt text](Images/Project_settings.png?raw=true "Project settings for the XeSS plugin")

<div style="page-break-after: always;"></div>

## Project packaging

No additional steps are needed for packaging.

## Localization support

Localized text is supported via string table "XeSSStringTable", which can be referenced in widgets, Blueprint and C++ code.

Currently these cultures are supported:

Abbreviation | Name                  |
------------ | --------------------- |
ar-SA        | Arabic (Saudi Arabia) |
da-DK        | Danish (Denmark)      |
de-DE        | German (Germany)      |
en           | English               |
es-ES        | Spanish (Spain)       |
fr-FR        | French (France)       |
it-IT        | Italian (Italy)       |
ja-JP        | Japanese (Japan)      |
ko-KR        | Korean (South Korea)  |
nl-NL        | Dutch (Netherlands)   |
pl-PL        | Polish (Poland)       |
pt-PT        | Portuguese (Portugal) |
ru-RU        | Russian (Russia)      |
uk-UA        | Ukrainian (Ukraine)   |
zh-Hans      | Chinese (Simplified)  |
zh-Hant      | Chinese (Traditional) |

<div style="page-break-after: always;"></div>

## Accessing the plugin in a game project

This plugin offers different ways for developers to access the underlying XeSS functionality.

### UE Console Commands

> It is more convenient to use for testing during development, not recommended for shipping.

#### XeSS-SR

To enable it:

```text
r.XeSS.Enabled 1
```

To change the Quality Mode:

```text
r.XeSS.Quality <quality_mode>
```

Where `<quality_mode>` represents the scale factor:

<!-- QUALITY EDIT: -->
Value | Quality Mode       | Scale factor |
----- | ------------------ | ------------ |
0     | Ultra Performance  | 3            |
1     | Performance        | 2.3          |
2     | Balanced (default) | 2            |
3     | Quality            | 1.7          |
4     | Ultra Quality      | 1.5          |
5     | Ultra Quality Plus | 1.3          |
6     | Anti-Aliasing      | 1            |

Auto exposure, which is enabled by default.

```text
r.XeSS.AutoExposure 1
```

> For more details on exposure calculation, please check `Intel® XeSS Developer Guide` in the `Documents` folder.

#### XeSS-FG

To enable it:

```text
r.XeFG.Enabled 1
```

To configure the maximum number of interpolated frames:

```text
r.XeFG.MaxInterpolatedFrames <value>
```

Where `<value>` is the number of frames XeFG interpolates between consecutive original frames.

* Valid range: `[1, r.XeFG.MaxInterpolatedFramesSupported]`
* Default: `1`

Examples:

* `1` = 2x frame rate (1 original + 1 interpolated frame)
* `2` = 3x frame rate (1 original + 2 interpolated frames)

To query the maximum number of interpolated frames supported:

```text
r.XeFG.MaxInterpolatedFramesSupported
```

This is a read-only console variable that shows the maximum interpolated frames supported by XeFG on the current platform.

#### XeLL

To enable it:

```text
r.XeLL.Enabled 1
```

### Blueprint API

Blueprint offers the highest level of compatibility and is the recommended way for shipping.

#### XeSS-SR Blueprint API

This plugin provides support for querying and settings XeSS-SR quality modes with Blueprint. It is recommended to use these functions when creating settings menus.

* _Is Intel(R) XeSS-SR Supported_
* _Get Supported Intel(R) XeSS-SR Quality Modes_
* _Get Current Intel(R) XeSS-SR Quality Mode_
* _Get Default Intel(R) XeSS-SR Quality Mode_
* _Set Intel(R) XeSS-SR Quality Mode_

![Alt text](Images/Blueprint_example.png?raw=true "Simple Blueprint example")

**Notes:**

1. It is recommend to use the _Get Default Intel(R) XeSS Quality Mode_ Blueprint function to set the out-of-the-box XeSS Quality Mode in the game's UI.
Based on the passed Screen Resolution the recommended Quality Mode will be returned - for resolutions with pixel counts corresponding to 1920x1080 and lower it will be _Balanced_, for higher resolutions it will be _Performance_.

2. For XeSS-SR, Blueprint API is recommended to use in Blueprint or C++, for plugin loses control to upscaling screen percentage directly since UE 5.1 and `r.ScreenPercentage` is set in Blueprint API to keep its backward compatibility.

#### XeSS-FG Blueprint API

* _Is Intel(R) XeSS-FG Supported_
* _Get Supported Intel(R) XeSS-FG Modes_
* _Get Current Intel(R) XeSS-FG Mode_
* _Set Intel(R) XeSS-FG Mode_
* _If Relaunch is Required by XeSS-FG_
* _Get Current Intel(R) XeSS-FG UI Composition State_
* _Set Intel(R) XeSS-FG UI Composition State_

#### XeLL Blueprint API

* _Is Intel(R) XeLL Supported_
* _Get Supported Intel(R) XeLL Modes_
* _Set Intel(R) XeLL Mode_
* _Get Current Intel(R) XeLL Mode_
* _Get Flash Indicator Enabled_
* _Get Game to Render Latency_
* _Get Game Latency_
* _Get Render Latency_
* _Get Simulation Latency_
* _Get Render Submit Latency_
* _Get Present Latency_
* _Get Input Latency_
* _Get Latency Mark Enabled_

### C++ API

#### XeSS-FG C++ API

Similar to `GAverageFPS`, `GXeFGAverageFPS` is offered to display FPS counter in-game, sample code fragment as follows:

```cpp
#include "XeFGRHI.h"

// Draw the FPS counter.
Canvas->DrawShadowedString(
    X,
    Y,
    *FString::Printf(TEXT("%5.2f FPS"), GXeFGAverageFPS),
    Font,
    FPSColor
);
```

<div style="page-break-after: always;"></div>

## Debugging

### Plugin log file

The XeSS plugin creates a separate log file `xess.log` in the same directory as Unreal Engine log files (typically `<project>/Saved/Logs/`). This log file is available in all build configurations including Shipping, and contains:

* Plugin version information
* SDK version information for XeSS-SR, XeSS-FG, and XeLL
* Additional diagnostic information

The log file is rewritten each time the application starts.

### Querying SDK version

Feature | Console Command  |
------- | ---------------- |
XeSS-SR | `r.XeSS.Version` |
XeSS-FG | `r.XeFG.Version` |
XeLL    | `r.XeLL.Version` |

### Querying feature support on current platform

Feature | Console Command    |
------- | ------------------ |
XeSS-SR | `r.XeSS.Supported` |
XeSS-FG | `r.XeFG.Supported` |
XeLL    | `r.XeLL.Supported` |

The support status depends on OS, RHI and [Unreal Engine version](#unreal-engine-versions-supported). Please check FAQ for more detailed information.

### Verifying if a feature is enabled in a running game

The easiest way to confirm that XeSS-SR or XeSS-FG are enabled is via:

```text
stat GPU
```

This will bring up real-time per-frame stats. `XeSS` or `XeFG` should be visible as one of the rendering passes.

For XeSS-SR, a dedicated stat console command is offered:

```text
stat XeSS
```

You can use it to check:

* The number of XeSS contexts being used (especially useful in split screen mode)
* GPU memory usage for temporary buffer storage
* GPU memory usage for temporary texture storage

For XeSS-FG, a dedicated stat console command is offered:

```text
stat XeFG
```

You can use it to check average FPS and frame count presented with XeSS-FG.

![Alt text](Images/StatXeFG.png?raw=true "Stat XeFG screenshot")

### Creating frame dumps

Frame dump console variables have been deprecated, please use [Intel® XeSS Inspector](https://intel.com/xess-inspector) instead.

### Using the High Resolution Screenshot tool

UE provides a console command that allows to take high resolution screen captures:

```text
HighResShot <screen_resolution>
```

When using this tool while XeSS-SR is engaged please make sure to set _screen_resolution_ to the currently set output resolution (can be set with _r.SetRes <screen_resolution>_).
Otherwise, the capture tool will change the target resolution, which will re-initialize the XeSS context and drop all temporally accumulated data. In result the captured image will not reflect the actual quality seen on the screen.

<div style="page-break-after: always;"></div>

## FAQ

### Platforms supported

Currently, only Windows x64 is supported.

### Rendering Hardware Interfaces (RHIs) supported

Feature | RHIs Supported |
------- | -------------- |
XeSS-SR | DirectX 12     |
XeSS-FG | DirectX 12     |
XeLL    | DirectX 12     |

### Unreal Engine versions supported

Feature | UE Versions Supported    |
------- | ------------------------ |
XeSS-SR | 4.26 and above           |
XeSS-FG | 5.2 and above, 4.27-5.1* |
XeLL    | 4.27 and above           |

> *) For XeSS-FG with UE 4.27 and 5.0, 5.1, UE source code patch is required.
>
> 1. Contact your Intel representative to obtain the source code patch.
>
> 1. Apply the patch to the engine source code.
>
> 1. Modify `XeSSCommonMacros.h` with the following code if it is not defined in your source patch:
>
> ```text
> #define XESS_ENGINE_WITH_XEFG_PATCH 1
> ```

### Relationship between XeSS-FG and XeLL

XeSS-FG requires XeLL to function. Enabling XeSS-FG will also enable XeLL, but disabling XeSS-FG will not disable XeLL, which is intentional.

### Compatibility with other vendors' frame generation plugins

For XeSS-SR and XeLL, there should be no conflict.

For XeSS-FG, only one plugin could take effect due to swap chain override being required. Real-time switching between frame generation techniques is not supported.
You can disable it via following options and restart the game to make another plugin work.

1. To set the console variable `r.XeFG.OverrideSwapChain` to `0` via a ini file. e.g., add the following lines to the `Engine.ini` file:

   ```ini
   [SystemSettings]
   r.XeFG.OverrideSwapChain=0
   ```

2. Start the game with the command line option `-XeFGOverrideSwapChainDisabled`.

   > **Note**: Using this command line option will also set the console variable `r.XeFG.OverrideSwapChain` to `0`.

### Performance is not as good as expected on Intel discrete GPUs

It could be due to incorrect BIOS settings. Please ensure the following configurations are applied:

* Compatibility Support Module (CSM) or Legacy Mode: **Disabled**
* UEFI Boot Mode: **Enabled**
* The following settings must be **Enabled** (or set to **Auto** if the **Enabled** option is unavailable):
  * Above 4G Decoding
  * Resizable BAR Support

You can use the [Intel® Driver and Support Assistant](https://www.intel.com/content/www/us/en/support/intel-driver-support-assistant.html) to verify if Resizable BAR Support is enabled.

**Important Note:**

Resizable BAR is not the only necessary BIOS setting; all of the above configurations are required. If any are improperly set, it may negatively impact GPU performance.

For more details, visit [Intel's Support Article](https://www.intel.com/content/www/us/en/support/articles/000090831/graphics.html).

### FPS drops after enabling XeSS-FG, what's wrong?

It may happen if you check FPS with `stat FPS` console command, because XeSS SDK calls additional `Present` API, which is not counted by `stat FPS`.
To check the real FPS number, you could use console command `stat XeFG` or use Windows Game Bar via pressing Windows logo key + G.

<div style="page-break-after: always;"></div>

## Known issues

* Pre-built UE 5.1 package doesn't work with 5.1.0, please upgrade to 5.1.1.

* If following link error occurs with pre-built packages, please upgrade Visual Studio to the latest version.

```text
error LNK2019: unresolved external symbol __std_find_trivial_4 referenced in function "int * __cdecl __std_find_trivial<int,char>(int *,int *,char)" (??$__std_find_trivial@HD@@YAPEAHPEAH0D@Z)
```

* Pre-exposure is still not used in the exposure calculation process.

* XeSS-SR in Vulkan may not function properly on non-Intel GPUs.

* XeLL and XeSS-FG don't support Editor yet. Please use Standalone mode.

* XeSS-FG doesn't support fullscreen exclusive mode and will be disabled in that mode.

* XeSS-FG doesn't support split screen and Virtual Reality (VR) due to XeSS SDK API limitations.

* XeSS-SR has a known resource leak issue with split screen and Virtual Reality (VR) support in Unreal Engine 4. A UE source patch is required to address this issue. Please contact your Intel representative to obtain the patch.
