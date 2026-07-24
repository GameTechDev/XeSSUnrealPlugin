# Intel® XeSS Plugin for Unreal Engine


## Table of Contents

- [Introduction](#introduction)
- [Downloads](#downloads)
- [Installing the plugin](#installing-the-plugin)
- [Enabling the plugin in the Editor](#enabling-the-plugin-in-the-editor)
- [Project settings](#project-settings)
- [Project packaging](#project-packaging)
- [Localization support](#localization-support)
- [Accessing the plugin in a game project](#accessing-the-plugin-in-a-game-project)
  - [UE Console Commands](#ue-console-commands)
  - [Blueprint API](#blueprint-api)
  - [C++ API](#c-api)
- [Debugging](#debugging)
- [FAQ](#faq)
- [Known issues](#known-issues)

<div style="page-break-after: always;"></div>

## Introduction

Intel® XeSS Plugin for Unreal Engine integrates Intel® [XeSS 3 technologies](https://www.intel.com/content/www/us/en/developer/topic-technology/gamedev/xess.html) into Unreal Engine (UE). XeSS Super Resolution (XeSS-SR), XeSS Frame Generation (XeSS-FG) including Multi-Frame Generation and Xe Low Latency (XeLL) are parts of XeSS 3.

For more information, please visit: [https://github.com/intel/xess](https://github.com/intel/xess).

If you encounter any integration issues, feel free to use the [Intel® XeSS Inspector](https://intel.com/xess-inspector). This tool is specifically designed to simplify the validation and debugging of XeSS integration within applications.

## Downloads

Please open the [Releases](https://github.com/GameTechDev/XeSSUnrealPlugin/releases) page to download compiled plugin packages.

## Installing the plugin

> **Note:** For C++ Unreal projects, it is recommended to copy the plugin to `<PROJECT_DIR>/Plugins/XeSS` instead.
> No standalone build step is required — the plugin will be built along with your project build.
> The plugin will be enabled automatically; no manual enabling in the Editor is needed.
> After copying, regenerate Visual Studio project files (right-click the `.uproject` file and select **Generate Visual Studio project files**).

For Blueprint Unreal projects or engine-level installation, follow the steps below.

### Step 1: Build the plugin (for the source code version of UE only)

- Open the terminal and go to `<UE_ROOT_DIR>/Engine/Build/BatchFiles`

- Run the command line:

   <!-- Use ```text (not ```bat) as a workaround to generate PDF file -->

   ```text
   RunUAT.bat BuildPlugin -Plugin="<DOWNLOADED_PLUGIN_DIR>/XeSS.uplugin" -Package="<DESTINATION_PLUGIN_DIR>" -TargetPlatforms=Win64 -VS2019
   ```

   > **Note:** The last parameter is the version of Visual Studio used to build, required by UE 4 only.

### Step 2: Copy built files to the engine plugins directory

Copy the (pre-)built plugin files to the appropriate location:

- For UE 4: `<UE_ROOT_DIR>/Engine/Plugins/Runtime/Intel/XeSS`

- For UE 5: `<UE_ROOT_DIR>/Engine/Plugins/Marketplace/XeSS`
  > **Note:** Please maintain this exact folder structure when installing the plugin. Deviating from this path may cause failures when packaging your game from the Editor. If folder `Marketplace` doesn't exist, please create it manually.

<div style="page-break-after: always;"></div>

## Enabling the plugin in the Editor

To enable the XeSS plugin for a project, navigate to the plugin settings.

![UE 4 project settings](Images/Plugin_settings.png?raw=true "UE 4 project settings")

Use the search box to find the XeSS plugin and ensure it is enabled.

![Enable XeSS plugin](Images/Enabled_xess_plugin.png?raw=true "Enable XeSS plugin")

<div style="page-break-after: always;"></div>

## Project settings

Once this plugin is enabled, its default behavior can be modified in the Project Settings menu.

It is possible to disable/enable the plugin in the Editor viewports.

![Project settings for the XeSS plugin](Images/Project_settings.png?raw=true "Project settings for the XeSS plugin")

<div style="page-break-after: always;"></div>

## Project packaging

No additional steps are needed for packaging.

## Localization support

Localized text is supported via the string table "XeSSStringTable", which can be referenced in widgets, Blueprint, and C++ code.

Currently, these cultures are supported:

| Abbreviation | Name                       |
| ------------ | -------------------------- |
| ar-SA        | Arabic (Saudi Arabia)      |
| da-DK        | Danish (Denmark)           |
| de-DE        | German (Germany)           |
| en           | English                    |
| es-ES        | Spanish (Spain)            |
| fr-FR        | French (France)            |
| it-IT        | Italian (Italy)            |
| ja-JP        | Japanese (Japan)           |
| ko-KR        | Korean (South Korea)       |
| nl-NL        | Dutch (Netherlands)        |
| pl-PL        | Polish (Poland)            |
| pt-PT        | Portuguese (Portugal)      |
| ru-RU        | Russian (Russia)           |
| uk-UA        | Ukrainian (Ukraine)        |
| zh-Hans      | Chinese (Simplified)       |
| zh-Hant      | Chinese (Traditional)      |

<div style="page-break-after: always;"></div>

## Accessing the plugin in a game project

This plugin offers different ways for developers to access the underlying XeSS functionality.

### UE Console Commands

> **Note:** Console commands are more convenient for testing during development, but are not recommended for shipping.

#### XeSS-SR

To enable XeSS-SR:

```text
r.XeSS.Enabled 1
```

To change the Quality Mode:

```text
r.XeSS.Quality <QUALITY_MODE>
```

Where `<QUALITY_MODE>` represents the scale factor:

<!-- QUALITY EDIT: -->

| Value | Quality Mode        | Scale factor |
| ----- | ------------------- | ------------ |
| 0     | Ultra Performance   | 3            |
| 1     | Performance         | 2.3          |
| 2     | Balanced (default)  | 2            |
| 3     | Quality             | 1.7          |
| 4     | Ultra Quality       | 1.5          |
| 5     | Ultra Quality Plus  | 1.3          |
| 6     | Anti-Aliasing       | 1            |

> **Note (UE 5.1+):** When using console commands, `r.ScreenPercentage` must also be set to match the selected quality mode. The required value is `100 / scale_factor` (e.g., `50` for Balanced with scale factor 2). You can read the optimal value from the read-only CVar `r.XeSS.OptimalScreenPercentage` after setting `r.XeSS.Quality`. In UE versions prior to 5.1, the plugin manages the screen percentage automatically via the `ICustomStaticScreenPercentage` interface and this step is not needed. The [Blueprint API](#blueprint-api) handles this automatically for all engine versions.

Auto exposure is enabled by default:

```text
r.XeSS.AutoExposure 1
```

> **Note:** For more details on exposure calculation, please check the `Intel® XeSS Developer Guide` in the `Documents` folder.

#### XeSS-FG

To enable XeSS-FG:

```text
r.XeFG.Enabled 1
```

To configure the maximum number of interpolated frames:

```text
r.XeFG.MaxInterpolatedFrames <VALUE>
```

Where `<VALUE>` is the number of frames XeFG interpolates between consecutive original frames.

- Valid range: `0` (auto) or `[1, r.XeFG.MaxInterpolatedFramesSupported]`
- Default: `0`

Examples:

- `0` = auto (uses `r.XeFG.MaxInterpolatedFramesSupported`)
- `1` = 2x frame rate (1 original + 1 interpolated frame)
- `2` = 3x frame rate (1 original + 2 interpolated frames)

To query the maximum number of interpolated frames supported:

```text
r.XeFG.MaxInterpolatedFramesSupported
```

This is a read-only console variable that shows the maximum interpolated frames supported by XeFG on the current platform.

To configure the XeSS-FG UI composition state:

```text
r.XeFG.UICompositionState <VALUE>
```

Where `<VALUE>` controls whether XeSS-FG UI composition is enabled:

| Value | UI composition state |
| ----- | -------------------- |
| 0     | Disabled (default)   |
| 1     | Enabled              |

If something is wrong with your UI, try setting this console variable to `1`.

#### XeLL

To enable XeLL:

```text
r.XeLL.Enabled 1
```

### Blueprint API

Blueprint offers the highest level of compatibility and is the recommended way for shipping.

#### XeSS-SR Blueprint API

This plugin provides support for querying and setting XeSS-SR quality modes with Blueprint. It is recommended to use these functions when creating settings menus.

- _Is Intel(R) XeSS-SR Supported_
- _Get Supported Intel(R) XeSS-SR Quality Modes_
- _Get Current Intel(R) XeSS-SR Quality Mode_
- _Get Default Intel(R) XeSS-SR Quality Mode_
- _Set Intel(R) XeSS-SR Quality Mode_

![Simple Blueprint example](Images/Blueprint_example.png?raw=true "Simple Blueprint example")

**Notes:**

1. It is recommended to use the _Get Default Intel(R) XeSS Quality Mode_ Blueprint function to set the out-of-the-box XeSS Quality Mode in the game's UI. Based on the passed Screen Resolution, the recommended Quality Mode will be returned - for resolutions with pixel counts corresponding to 1920x1080 and lower it will be _Balanced_, for higher resolutions it will be _Performance_.

2. For XeSS-SR, the Blueprint API is recommended for use in Blueprint or C++. The plugin loses control of upscaling screen percentage directly since UE 5.1, and `r.ScreenPercentage` is set in the Blueprint API to maintain backward compatibility.

#### XeSS-FG Blueprint API

- _Is Intel(R) XeSS-FG Supported_
- _Get Supported Intel(R) XeSS-FG Modes_
- _Get Current Intel(R) XeSS-FG Mode_
- _Set Intel(R) XeSS-FG Mode_
- _If Relaunch is Required by XeSS-FG_
- _Get Current Intel(R) XeSS-FG UI Composition State_
- _Set Intel(R) XeSS-FG UI Composition State_

#### XeLL Blueprint API

- _Is Intel(R) XeLL Supported_
- _Get Supported Intel(R) XeLL Modes_
- _Set Intel(R) XeLL Mode_
- _Get Current Intel(R) XeLL Mode_
- _Get Flash Indicator Enabled_
- _Get Game to Render Latency_
- _Get Game Latency_
- _Get Render Latency_
- _Get Simulation Latency_
- _Get Render Submit Latency_
- _Get Present Latency_
- _Get Input Latency_
- _Get Latency Mark Enabled_

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

The XeSS plugin creates a separate log file `xess.log` in the same directory as UE log files (typically `<PROJECT_DIR>/Saved/Logs/`). This log file is available in all build configurations including Shipping, and contains:

- Plugin version information
- SDK version information for XeSS-SR, XeSS-FG, and XeLL
- Additional diagnostic information

The log file is rewritten each time the application starts.

### Querying SDK version

| Feature | Console Command  |
| ------- | ---------------- |
| XeSS-SR | `r.XeSS.Version` |
| XeSS-FG | `r.XeFG.Version` |
| XeLL    | `r.XeLL.Version` |

### Querying feature support on current platform

| Feature | Console Command     |
| ------- | ------------------- |
| XeSS-SR | `r.XeSS.Supported`  |
| XeSS-FG | `r.XeFG.Supported`  |
| XeLL    | `r.XeLL.Supported`  |

The support status depends on OS, RHI, and [UE version](#ue-versions-supported). Please check the FAQ for more detailed information.

### Verifying if a feature is enabled in a running game

The easiest way to confirm that XeSS-SR or XeSS-FG is enabled is via:

```text
stat GPU
```

This will bring up real-time per-frame stats. `XeSS` or `XeFG` should be visible as one of the rendering passes.

For XeSS-SR, a dedicated stat console command is offered:

```text
stat XeSS
```

You can use it to check:

- The number of XeSS contexts being used (especially useful in split screen mode)
- GPU memory usage for temporary buffer storage
- GPU memory usage for temporary texture storage

For XeSS-FG, a dedicated stat console command is offered:

```text
stat XeFG
```

You can use it to check the average FPS and frame count presented with XeSS-FG.

![Stat XeFG screenshot](Images/StatXeFG.png?raw=true "Stat XeFG screenshot")

### Creating frame dumps

Frame dump console variables have been deprecated, please use [Intel® XeSS Inspector](https://intel.com/xess-inspector) instead.

### Using the High Resolution Screenshot tool

UE provides a console command that allows you to take high resolution screen captures:

```text
HighResShot <SCREEN_RESOLUTION>
```

When using this tool while XeSS-SR is engaged, please make sure to set `<SCREEN_RESOLUTION>` to the currently set output resolution (can be set with `r.SetRes <SCREEN_RESOLUTION>`). Otherwise, the capture tool will change the target resolution, which will re-initialize the XeSS context and drop all temporally accumulated data. As a result, the captured image will not reflect the actual quality seen on the screen.

<div style="page-break-after: always;"></div>

## FAQ

### Platforms supported

Currently, only Windows x64 is supported.

### Rendering Hardware Interfaces (RHIs) supported

| Feature  | RHIs Supported                   |
| -------- | -------------------------------- |
| XeSS-SR  | DirectX 11*, DirectX 12, Vulkan* |
| XeSS-FG  | DirectX 12                       |
| XeLL     | DirectX 12                       |

> **Note:** Items marked with * have limitations: DirectX 11 is supported only on Intel® Arc™ Graphics or later, and for Vulkan limitations, see the [Known issues](#known-issues) section.

### UE versions supported

| Feature | UE Versions Supported    |
| ------- | ------------------------ |
| XeSS-SR | 4.26 and above           |
| XeSS-FG | 5.2 and above, 4.27-5.1* |
| XeLL    | 4.27 and above           |

> **Note:** For XeSS-FG with UE 4.27, 5.0, and 5.1, a UE source code patch is required.
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

For XeSS-FG, only one plugin can take effect due to swap chain override being required. Real-time switching between frame generation techniques is not supported. You can disable it via the following options and restart the game to make another plugin work:

1. Set the console variable `r.XeFG.OverrideSwapChain` to `0` via an ini file. For example, add the following lines to the `Engine.ini` file:

   ```ini
   [SystemSettings]
   r.XeFG.OverrideSwapChain=0
   ```

2. Start the game with the command line option `-XeFGOverrideSwapChainDisabled`.

   > **Note:** Using this command line option will also set the console variable `r.XeFG.OverrideSwapChain` to `0`.

### Performance is not as good as expected on Intel discrete GPUs

It could be due to incorrect BIOS settings. Please ensure the following configurations are applied:

- Compatibility Support Module (CSM) or Legacy Mode: **Disabled**
- UEFI Boot Mode: **Enabled**
- The following settings must be **Enabled** (or set to **Auto** if the **Enabled** option is unavailable):
  - Above 4G Decoding
  - Resizable BAR Support

You can use the [Intel® Driver and Support Assistant](https://www.intel.com/content/www/us/en/support/intel-driver-support-assistant.html) to verify if Resizable BAR Support is enabled.

**Important Note:**

Resizable BAR is not the only necessary BIOS setting; all of the above configurations are required. If any are improperly set, it may negatively impact GPU performance.

For more details, visit [Intel's Support Article](https://www.intel.com/content/www/us/en/support/articles/000090831/graphics.html).

### FPS drops after enabling XeSS-FG

This may happen if you check FPS with the `stat FPS` console command, because the XeSS SDK calls an additional `Present` API, which is not counted by `stat FPS`. To check the real FPS number, you can use the console command `stat XeFG` or use Windows Game Bar by pressing Windows logo key + G.

<div style="page-break-after: always;"></div>

## Known issues

- Pre-built UE 5.1 package doesn't work with 5.1.0, please upgrade to 5.1.1.

- XeLL doesn't work with UE 5.8.0; please wait for an Unreal Engine update or contact your Intel representative to obtain a UE source patch.

- If the following link error occurs with pre-built packages, please upgrade Visual Studio to the latest version:

  ```text
  error LNK2019: unresolved external symbol __std_find_trivial_4 referenced in function "int * __cdecl __std_find_trivial<int,char>(int *,int *,char)" (??$__std_find_trivial@HD@@YAPEAHPEAH0D@Z)
  ```

- Pre-exposure is still not used in the exposure calculation process.

- XeSS-SR in Vulkan may not function properly on non-Intel GPUs.

- XeLL and XeSS-FG don't support Editor yet. Please use Standalone mode.

- XeSS-FG doesn't support fullscreen exclusive mode and will be disabled in that mode.

- XeSS-FG doesn't support split screen and Virtual Reality (VR) due to XeSS SDK API limitations.

- XeSS-SR has a known resource leak issue with split screen and Virtual Reality (VR) support in UE 4. A UE source patch is required to address this issue. Please contact your Intel representative to obtain the patch.

- XeSS-FG may cause a crash when used alongside certain older versions of [OBS (Open Broadcaster Software)](https://obsproject.com). To resolve this issue, upgrade OBS to version 30.1.0 or above.
