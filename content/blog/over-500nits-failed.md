---
title: "Trying to get past the 500 nits limit of the MacBook Pro (and failing)"
subtitle: "Investigating why the new MacBook Pro XDR display is capped at 500 nits, despite being advertised as '1000 nits sustained'"
excerpt: |-
    The 2021 MacBook Pro came with a greatly improved miniLED XDR display which is advertised as being able to handle 1000 nits sustained brightness.

    Contrary to everyone's expectations, when the machine got into the hands of users, it was clear that the display was still capped at the previous 500 nits for normal usage.

    My attempt at trying to break through that limit failed, but the details might be useful for someone more determined. 
description: An investigation into why the new MacBook Pro XDR display is capped at 500 nits, despite being advertised as '1000 nits sustained brightness'.
date: 2022-02-04T19:26:36+02:00
author: Alin Panaitiu
draft: false
image: over-500nits-failed.png
images:
-  /images/over-500nits-failed/768_over-500nits-failed.png
tags:
    - macbook
    - macbook pro
    - nits
    - nits limit
    - hdr
    - maximum brightness
    - lunar
categories: 
    - macOS reverse engineering

pageStyles: 
  - file: article.sass
    media: "(prefers-color-scheme: light), (prefers-color-scheme: no-preference)"
  - file: article-dark.sass
    media: "(prefers-color-scheme: dark)"
---

Exactly 3 months and a day after placing an order through a Romanian Apple reseller, I finally got my 14-inch M1 Max.

*Well, actually.. I first got the wrong configuration (base model instead of CTO), had to return it to them after wasting a day on migrating my data to it, they sent my money back by mistake, had to pay them again, and after many calls and emails later the correct laptop arrived.*

{{< img src="m1max.png" alt="M1 Max MacBook Pro box" sizes="(min-width: 60em) 90%, 90vw" >}}

As soon as these devices were in the hands of users, requests started coming in for **[Lunar](https://lunar.fyi)** to provide an option to [get past the 500 nits limit for everyday usage](https://github.com/alin23/Lunar/issues/417)

Over the last week I tried my best to figure out how to do this, but it's either impossible to raise the nits limit from userspace, or I just don't have the necessary expertise.

I'll share some details that I found while reverse engineering my way through the macOS part that handles brightness.

## Testing the system

### Playing a HDR video

I first started by playing this HDR test video *(open it in latest Chrome or Safari for best results)*: [hdr-test-pattern.webm](https://files.alinpanaitiu.com/hdr-test-pattern.webm)

Which resulted in a blinding white at 1600 nits:
{{< img src="hdr-result.png" alt="HDR white being whiter than the webpage white" sizes="(min-width: 60em) 90%, 90vw" >}}

This generated the following logs in Console.app:

```swift
WindowServer    Display 1 setting nits to 888.889
corebrightnessd SDR - perceptual ramp clocked: 227.095169 -> 252.268112 - 49.169426% (239.142059 Nits)
WindowServer    Display 1 commitBrightness sdr: 211.603, headroom: 4.20075, ambient: 4.3396, filtered ambient: 13.6333, limit: 1600
```

----

### SDR cap in normal lighting

After setting the display brightness to max, I could see in the logs that **SDR** (Standard Dynamic Range) was being capped at **400 nits**:

```swift
WindowServer    Display 1 setting nits to 1600
WindowServer    Display 1 setting display headroom hint to 7.56866
WindowServer    Display 1 commitBrightness sdr: 216.548, headroom: 7.38865, ambient: 4.24854, filtered ambient: 13.3472, limit: 1600
corebrightnessd PCC: Set PCC: Factor:=1.0496 CabalFactor:=0.0033 time=2.000000 Lux:=13.6080 Nits:=229.1757 result=1 error=(null)
WindowServer    Display 1 commitBrightness sdr: 301.188, headroom: 5.3123, ambient: 4.24854, filtered ambient: 13.3472, limit: 1600
WindowServer    Display 1 setting nits to 1602.03
corebrightnessd levelPercentage 0.334298, level = 4.967383 (nits/pwm), lux = 15.000000
WindowServer    Display 1 commitBrightness sdr: 301.571, headroom: -1, ambient: 4.79275, filtered ambient: 15.0569, limit: -1
WindowServer    Display 1 setting display headroom hint to 5.27556
WindowServer    Display 1 commitBrightness sdr: 321.478, headroom: 4.97701, ambient: 4.79275, filtered ambient: 15.0569, limit: 1600
WindowServer    Display 1 commitBrightness sdr: 340.675, headroom: 4.69655, ambient: 4.79275, filtered ambient: 15.0569, limit: 1600
WindowServer    Display 1 commitBrightness sdr: 377.322, headroom: 4.24041, ambient: 4.79275, filtered ambient: 15.0569, limit: 1600
corebrightnessd PCC: Set PCC: Factor:=1.0340 CabalFactor:=0.0023 time=2.000000 Lux:=15.0569 Nits:=377.3223 result=1 error=(null)
WindowServer    Display 1 setting nits to 1600
WindowServer    Display 1 setting display headroom hint to 4
WindowServer    Display 1 commitBrightness sdr: 400, headroom: -1, ambient: 4.96577, filtered ambient: 15.6004, limit: -1

```

{{< img src="hdr-console.png" alt="HDR white and console logs side by side" sizes="(min-width: 60em) 90%, 90vw" >}}

----

### SDR cap in direct sunlight

Shining a flashlight directly into the Ambient Light Sensor allowed **SDR** to jump up to **500 nits**:

```swift
WindowServer    Display 1 commitBrightness sdr: 400, headroom: -1, ambient: 322.204, filtered ambient: 1012.24, limit: -1
WindowServer    Display 1 commitBrightness sdr: 400.484, headroom: 1, ambient: 322.204, filtered ambient: 1012.24, limit: 400.484
WindowServer    Display 1 setting nits to 400.484
WindowServer    Display 1 commitBrightness sdr: 401.15, headroom: 1, ambient: 322.204, filtered ambient: 1012.24, limit: 401.15
WindowServer    Display 1 setting nits to 401.15
WindowServer    Display 1 commitBrightness sdr: 401.223, headroom: 1, ambient: 322.204, filtered ambient: 1012.24, limit: 401.224
WindowServer    Display 1 setting nits to 401.223
WindowServer    Display 1 commitBrightness sdr: 401.223, headroom: -1, ambient: 370.814, filtered ambient: 1164.95, limit: -1
WindowServer    Display 1 commitBrightness sdr: 401.552, headroom: 1, ambient: 370.814, filtered ambient: 1164.95, limit: 401.552
corebrightnessd PCC: Set PCC: Factor:=1.7464 CabalFactor:=0.0498 time=2.000000 Lux:=1164.9467 Nits:=401.5517 result=1 error=(null)
WindowServer    Display 1 setting nits to 401.552
WindowServer    Display 1 commitBrightness sdr: 402.219, headroom: 1, ambient: 370.814, filtered ambient: 1164.95, limit: 402.219
WindowServer    Display 1 setting nits to 402.219
WindowServer    Display 1 commitBrightness sdr: 402.885, headroom: 1, ambient: 370.814, filtered ambient: 1164.95, limit: 402.885
WindowServer    Display 1 setting nits to 402.885
... lots of similar logs ...
WindowServer    Display 1 setting nits to 495.458
WindowServer    Display 1 commitBrightness sdr: 496.125, headroom: 1, ambient: 810.176, filtered ambient: 2545.24, limit: 496.125
WindowServer    Display 1 setting nits to 496.125
WindowServer    Display 1 commitBrightness sdr: 496.791, headroom: 1, ambient: 810.176, filtered ambient: 2545.24, limit: 496.792
WindowServer    Display 1 setting nits to 496.791
WindowServer    Display 1 commitBrightness sdr: 497.458, headroom: 1, ambient: 810.176, filtered ambient: 2545.24, limit: 497.458
WindowServer    Display 1 setting nits to 497.458
WindowServer    Display 1 commitBrightness sdr: 498.125, headroom: 1, ambient: 810.176, filtered ambient: 2545.24, limit: 498.125
WindowServer    Display 1 setting nits to 498.125
WindowServer    Display 1 commitBrightness sdr: 498.791, headroom: 1, ambient: 810.176, filtered ambient: 2545.24, limit: 498.792
WindowServer    Display 1 setting nits to 498.791
WindowServer    Display 1 commitBrightness sdr: 499.458, headroom: 1, ambient: 810.176, filtered ambient: 2545.24, limit: 499.458
WindowServer    Display 1 setting nits to 499.458
WindowServer    Display 1 commitBrightness sdr: 500, headroom: 1, ambient: 810.176, filtered ambient: 2545.24, limit: 500
WindowServer    Display 1 setting nits to 500
WindowServer    Display 1 commitBrightness sdr: 500, headroom: -1, ambient: 987.858, filtered ambient: 3103.45, limit: -1
```


## Dissecting the system

Since Big Sur, macOS transitioned from having the frameworks on the disk as separate binaries, to having a single file containing all the system libraries, called a `dyld_shared_cache`.

> * New in macOS Big Sur 11.0.1, the system ships with a built-in dynamic linker cache of all system-provided libraries. As part of this change, copies of dynamic libraries are no longer present on the filesystem. Code that attempts to check for dynamic library presence by looking for a file at a path or enumerating a directory will fail. Instead, check for library presence by attempting to dlopen() the path, which will correctly check for the library in the cache. (62986286)

Searching for keywords from the above logs surfaced only the dyld cache as expected.

{{< img src="dyld-cache-rg-result.png" alt="searching for nits in system" sizes="(min-width: 60em) 90%, 90vw" >}}

I used [dyld-shared-cache-extractor](https://github.com/keith/dyld-shared-cache-extractor) to drop the separate binaries on disk, then did another search there.

This surfaced up `QuartzCore` as the single place where that string could be found.

{{< img src="dyld-cache-extracted-rg-result.png" alt="searching for nits in extracted dyld cache" sizes="(min-width: 60em) 90%, 90vw" >}}

----

### Trying to abuse QuartzCore

After looking through the QuartzCore binary with Ghidra and finding some iOS headers for it on [limneos.net](https://developer.limneos.net/index.php?ios=15.2.1&framework=QuartzCore.framework&header=CAWindowServer.h), I created a sample Swift project to try to use some of the exported functions from it: [monitorpanel - main.swift](https://github.com/alin23/monitorpanel/blob/nits-limit/Sources/monitorpanel/main.swift)

Based on some open-sourced iOS jailbreak tweaks, I noticed that developers used the `CAWindowServer` class to interface with the display and HID components directly. The class was available here so I tried to do the same on macOS.

Unfortunately, `CAWindowServer.serverIfRunning` always returns `nil` and while `CAWindowServer.server(withOptions: nil)` returns a seemingly valid server, all external displays are forcefully disconnected when that server is created.

Using the below code, I succeeded in producing the `commitBrightness` log line in Console, but nothing really changed.

**code from [main.swift](https://github.com/alin23/monitorpanel/blob/nits-limit/Sources/monitorpanel/main.swift)**
```swift
func setToMax(_ d: CAWindowServerDisplay) {
    d.setBrightnessLimit(1600)
    d.setHeadroom(1)
    d.maximumBrightness = 1000.0
    d.setSDRBrightness(600)
    d.maximumHDRLuminance = 1600
    d.maximumReferenceLuminance = 1600
    d.maximumSDRLuminance = 1000
    d.contrast = 1.1
    d.commitBrightness(1)
    // d.update() // segfault
}

let ws: CAWindowServer? = (CAWindowServer.server(withOptions: nil) as? CAWindowServer) // disconnects external displays
if let ws = ws,
   let displays = ws.displays as? [CAWindowServerDisplay],
   let d = displays.first(where: { $0.deviceName == "primary" })
{
    setToMax(d)
}
```

**`commitBrightness` log line**

```swift
monitorpanel    Display 1 commitBrightness sdr: 600, headroom: 1, ambient: -1, filtered ambient: -1, limit: 1600
```

### CoreBrightness
While looking through Ghidra, I noticed that `QuartzCore` finally calls into `CoreBrightness` functions to increase the nits limit, so I took a look at the exported symbols on that binary.

Unfortunately, all the possibly useful symbols are not exported and trying to link against them would result in the `undefined symbols` error.

Adding the private symbols in the [CoreBrightness.tbd](https://github.com/alin23/monitorpanel/blob/nits-limit/Sources/CoreBrightness.framework/CoreBrightness.tbd) file doesn't help in this case.

```objc
// Uninteresting Exported Symbols

_OBJC_CLASS_$_BrightnessSystem
_OBJC_CLASS_$_BrightnessSystemClient
_OBJC_CLASS_$_BrightnessSystemClientInternal
_OBJC_CLASS_$_CBAdaptationClient
_OBJC_CLASS_$_CBBlueLightClient
_OBJC_CLASS_$_CBClient
_OBJC_CLASS_$_CBKeyboardPreferencesManager
_OBJC_CLASS_$_CBTrueToneClient
_OBJC_CLASS_$_DisplayServicesClient
_OBJC_CLASS_$_KeyboardBrightnessClient


// Interesting Not Exported Symbols

-[CBBrightnessProxySKL brightnessNotificationRequestEDR]
-[CBBrightnessProxySKL brightnessRequestEDRHeadroom]
-[CBBrightnessProxySKL brightnessRequestRampDuration]
-[CBBrightnessProxySKL commitBrightness:]
-[CBBrightnessProxySKL initWithSLSBrightnessControl:]
-[CBBrightnessProxySKL setAmbient:]
-[CBBrightnessProxySKL setBrightnessLimit:]
-[CBBrightnessProxySKL setHeadroom:]
-[CBBrightnessProxySKL setNotificationQueue:]
-[CBBrightnessProxySKL setPotentialHeadroom:]
-[CBBrightnessProxySKL setSDRBrightness:]
-[CBBrightnessProxySKL setWhitePoint:rampDuration:error:]
-[CBBrightnessProxySKL unregisterNotificationBlocks]
-[CBDisplayModuleSKL configureEDRSecPerStop]
-[CBDisplayModuleSKL configurePCCDefaults]
-[CBDisplayModuleSKL getBrightnessLimit]
-[CBDisplayModuleSKL getDynamicSliderAdjustedNits:]
-[CBDisplayModuleSKL getDynamicSliderAdjustedSDRNits]
-[CBDisplayModuleSKL getLinearBrightnessForNits:]
-[CBDisplayModuleSKL getLinearBrightness]
-[CBDisplayModuleSKL getMaxNitsAdjusted]
-[CBDisplayModuleSKL getMaxNitsEDR]
-[CBDisplayModuleSKL getMaxPanelNits]
-[CBDisplayModuleSKL getNitsForLinearBrightness:]
-[CBDisplayModuleSKL getNitsForUserBrightness:]
-[CBDisplayModuleSKL getPerceptualBrightness]
-[CBDisplayModuleSKL getSDRBrightnessCurrent]
-[CBDisplayModuleSKL getSDRBrightnessTarget:]
-[CBDisplayModuleSKL getSDRNitsCapped]
-[CBDisplayModuleSKL getUserBrightnessForNits:]
-[CBDisplayModuleSKL getUserBrightnessSloperExtended]
-[CBDisplayModuleSKL getUserBrightness]
-[CBDisplayModuleSKL handleBrightnessCapOverride:]
-[CBDisplayModuleSKL initialiseEDR]
-[CBDisplayModuleSKL initialiseSDR]
-[CBDisplayModuleSKL luminanceToPerceptual:]
-[CBDisplayModuleSKL panelMaxNitsOverride:]
-[CBDisplayModuleSKL perceptualToLuminance:]
-[CBDisplayModuleSKL rampDynamicSlider:withLength:]
-[CBDisplayModuleSKL rampEDRHedroom:withLength:]
-[CBDisplayModuleSKL rampFactor:withLength:]
-[CBDisplayModuleSKL rampManagerUpdateHandling]
-[CBDisplayModuleSKL rampNitsCap:]
-[CBDisplayModuleSKL rampSDRBrightness:withLength:properties:]
-[CBDisplayModuleSKL requestEDRHeadroomImmediate:]
-[CBDisplayModuleSKL requestEDRHeadroomTransition:withLength:]
-[CBDisplayModuleSKL requestEDRHeadroomTransitionStop]
-[CBDisplayModuleSKL requestFactorImmediate:]
-[CBDisplayModuleSKL requestFactorTransition:withLength:]
-[CBDisplayModuleSKL requestFactorTransitionStop]
-[CBDisplayModuleSKL requestSDRBrightnessTransition:]
-[CBDisplayModuleSKL requestSDRBrightnessTransition:withLength:properties:]
-[CBDisplayModuleSKL requestSDRBrightnessTransitionStop]
-[CBDisplayModuleSKL supportsDynamicSlider]
-[CBDisplayModuleSKL supportsEDR]
-[CBDisplayModuleSKL supportsSDRBrightness]
-[CBDisplayModuleSKL updateAmbient]
-[CBDisplayModuleSKL updateAutoBrightnessState:]
-[CBDisplayModuleSKL updateBrightnessState]
-[CBDisplayModuleSKL updateContrastEnhancerState:]
-[CBDisplayModuleSKL updateDynamicSliderAmbient]
-[CBDisplayModuleSKL updateDynamicSliderAutoBrightness]
-[CBDisplayModuleSKL updateDynamicSliderChargerState]
-[CBDisplayModuleSKL updateDynamicSliderScaler:]
-[CBDisplayModuleSKL updateEDRAmbient]
-[CBDisplayModuleSKL updateSDRBrightness:]
-[CBDisplayModuleSKL updateSDRNits:]
-[CBEDR appliedCompensation]
-[CBEDR availableHeadroom]
-[CBEDR brightnessCap]
-[CBEDR cappedHeadroomFromUncapped:]
-[CBEDR copyStatusInfo]
-[CBEDR description]
-[CBEDR initWithRampPolicy:potentialHeadroom:andReferenceHeadroom:]
-[CBEDR maxHeadroom]
-[CBEDR panelMax]
-[CBEDR referenceHeadroom]
-[CBEDR sanityCheck]
-[CBEDR sdrBrightness]
-[CBEDR secondsPerStop]
-[CBEDR setAppliedCompensation:]
-[CBEDR setBrightnessCap:]
-[CBEDR setPanelMax:]
-[CBEDR setSdrBrightness:]
-[CBEDR setSecondsPerStop:]
-[CBEDR shouldUpdateEDRForRequestedHeadroom:targetHeadroom:rampTime:]
-[CBEDR stopsFromHeadroomRatio:]
-[CBNVRAM backlightNitsDefault]
-[CBNVRAM backlightNitsMax]
-[CBNVRAM backlightNitsMin]
-[CBNVRAM dealloc]
-[CBNVRAM init]
-[CBNVRAM readBacklightNits]
-[CBNVRAM setBacklightNitsMax:]
-[CBNVRAM writeBacklightNits:]
```

### SkyLight
I knew from previous work on window management that the `SkyLight` framework is closely related to the WindowServer so I took a look at that too.

SkyLight exports a lot of symbols, and fortunately I had a good example on how to use them inside [yabai](https://github.com/koekeishiya/yabai/blob/master/src/misc/extern.h), a macOS window manager similar to i3 and bspwm.

But again, nothing useful is exported.

{{< img src="skylight-symbols.png" alt="Searching for nits in SkyLight" sizes="(min-width: 60em) 90%, 90vw" >}}

The function `kSLSBrightnessRequestEDRHeadroom` seemed promising but I always got a `SIGBUS` when trying to call it. I can't find its implementation so I don't know what parameters I should pass. I just guessed the first one could be a display ID.

```objc
@import Darwin;
@import Foundation;

// clang -fmodules -F/System/Library/PrivateFrameworks -framework SkyLight -o headroom headroom.m && ./headroom

extern int SLSMainConnectionID(void);
extern CFTypeRef SLSDisplayGetCurrentHeadroom(int did);
extern void kSLSBrightnessRequestEDRHeadroom(int did, CFTypeRef headroom);

const int MAIN_DISPLAY_ID = 1;

int main(int argc, char** argv)
{
    int cid = SLSMainConnectionID();
    NSLog(@"SLSMainConnectionID: %d", cid);

    CFTypeRef headroom = SLSDisplayGetCurrentHeadroom(MAIN_DISPLAY_ID);
    kSLSBrightnessRequestEDRHeadroom(MAIN_DISPLAY_ID, headroom);

    return 0;
}
```


## Other ideas

### Streaming to a dummy

While discussing this matter with István Tóth, the developer of [BetterDummy](https://github.com/waydabber/BetterDummy/), he came up with an interesting idea.

1. Create a `CGVirtualDisplay` with the same size as the built-in display
2. Tone map the SDR contents of the built-in display to 1000nits HDR video
3. `CGDisplayStream` that video to the virtual display 
4. Move the virtual display to the built-in display coordinates and use that as the main display

The streaming part already works in the [latest Beta of BetterDummy](https://github.com/waydabber/BetterDummy/releases/tag/v1.1.0-beta1) and seems pretty fast as well. But adding tone mapping might cause this to be too resource intensive to be used.

### Using private symbols

I think linking can be done against private symbols using memory offsets, I remember doing something like that 8 years ago at BitDefender, while trying to use the unexported `_decrypt` and `_generate_domain` methods of some DGA malware.

But the `dyld_shared_cache` model of macOS is something new to me and I don't have enough knowledge to be able to do that right now.

If someone has any idea how this can be achieved, I'd be glad if you could send me a hint through the [Contact page](/contact).