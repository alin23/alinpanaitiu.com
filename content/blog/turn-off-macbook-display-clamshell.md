---
title: "Reverse engineering the MacBook clamshell mode"
subtitle: "Closing the MacBook lid with an external monitor connected can turn off and disable the internal display. Let's figure out how macOS does that and bypass the lid sensors."
description: "Closing the MacBook lid with an external monitor connected can turn off and disable the internal display. Let's figure out how macOS does that and bypass the lid sensors."
excerpt: |-
    When using an external monitor, you might want to turn off the MacBook display to focus on the large screen.

    Closing the lid does that, but you lose TouchID, the webcam, the nice keyboard and trackpad. Surely there must be a way to call into that same code that lid closing does.
date: 2023-01-17T15:16:13+02:00
author: Alin Panaitiu
draft: false
image: /images/turn-off-macbook-display-clamshell.mp4
images:
-  https://img.panaitiu.com/_/og/plain/https%3A%2F%2Falinpanaitiu.com%2Fimages%2Fturn-off-macbook-display-clamshell.png@webp
tags:
    - macbook
    - macbook pro
    - clamshell
    - lid closed
    - lid open
    - disable screen
    - turn off display
categories:
    - macOS reverse engineering

pageStyles:
  - file: article.sass
    media: "(prefers-color-scheme: light), (prefers-color-scheme: no-preference)"
  - file: article-dark.sass
    media: "(prefers-color-scheme: dark)"
---

You just got a large, Ultrawide monitor for your MacBook. You hook it up and marvel at the amount of pixels.

You notice you never use the MacBook built-in display anymore, and it nags you to have it in your lower peripheral vision.

Closing the lid is not an option because you still use the keyboard and trackpad, maybe even the webcam and TouchID from time to time. So you try things:

- you try turning off the display by lowering the brightness completely. 🤔 hmm ok but now:
	- your mouse wanders to that screen sometimes
	- some windows get lost in there
	- ..and you still waste GPU cycles for rendering 6 million unused pixels
- you mirror the monitor to the built-in screen. nice, this solves the first two issues!
	- *ok but why did the resolution change? do I have to change it back every time I do this??*
	- *wait, why don't I get notifications anymore?! oh. there's [a setting for that](/images/notifications-mirroring.png)*
- you walk away from the desk, the screen goes to sleep
- you come back, the screen is now on something like 6% brightness, not completely off anymore
	- *ok, press `Brightness Down` again, I can live with that*
	- *oh, mirroring got disabled as well.. at least there's `Cmd`+`Brightness Down`*

Why isn't there a way to actually disable this screen?

## [BlackOut](https://lunar.fyi/#blackout)

Because a lot of users of my **[🌕 Lunar](https://lunar.fyi/)** app told me about their grievances with not being able to turn off individual displays in software, I went down the rabbit hole of display mirroring and automated all of the above.

{{< img src="blackout-ui.png" alt="Lunar interface showing the BlackOut function" sizes="(min-width: 60em) 810px, 90vw" >}}

Now someone can turn off and on any display at will using keyboard shortcuts, and can even automate the above **MacBook** + **monitor** workflow to trigger when an external monitor gets connected and disconnected.

But it's still nagging me that somehow macOS can actually disable the internal screen completely, but we're stuck with this *zero-brightness-mirroring* abomination.

## Clamshell Mode

When closing the MacBook lid while a monitor is still connected, the internal screen disappears from the screen list and the external monitors remain available.

This function is called *clamshell mode* in the laptop world. Congratulations, your $3000 all-in-one computer is now just an SoC with some USB-C ports. Ok, you also get the speakers and the inefficient cooling system.

{{< img src="macbook-clamshell-on-stand.jpg" alt="MacBook with lid closed sitting vertically on a wooden stand" sizes="(min-width: 60em) 810px, 90vw" >}}

In the pre-chunky-MacBook-Pro-with-notch era, the lid was detected as being closed using magnets in the lid, and some [hall effect sensors](https://guide-images.cdn.ifixit.com/igi/TWIoJgROcI65mq4A.full). So you were able to trick macOS into thinking the lid was closed by simply placing two powerful magnets at its sides.

With the new 2021 design, the MacBook has a [hinge sensor](https://www.ifixit.com/News/33952/apple-put-a-hinge-sensor-in-the-16-macbook-pro-what-could-it-be-for), that can detect not only if the lid is closed, but also the angle of its closing. Magnets can't trick'em anymore.

But all these sensors will probably just trigger some event in software, where a handler will decide if the display should be disabled or not, and call some `disableScreenInClamshellMode` function.

So where is that function, and can we call it ourselves?

## The software side

Since Apple Silicon, most userspace code lives in a single file called a DYLD Shared Cache. Since Ventura, that is located in a [Cryptex](https://eclecticlight.co/2022/11/16/cryptex-how-a-custom-iphone-is-changing-macos-updates/) (a read-only volume) at the following path:

`/System/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e`

Since that file is mostly an optimised concatenation of macOS Frameworks, we can extract the binaries using [keith/dyld-shared-cache-extractor](https://github.com/keith/dyld-shared-cache-extractor):

```sh
mkdir -p ~/Temp/dyld && cd ~/Temp/dyld
dyld-shared-cache-extractor /System/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e $PWD
```

Let's extract the exported and unexported symbols in text format to be able to search them easily using something like [ripgrep](https://github.com/BurntSushi/ripgrep).

I'm using `/usr/bin/nm` with [fd](https://github.com/sharkdp/fd)'s `-x` option to take advantage of parallelisation. I like its syntax more than `parallel`'s since it has integrated interpolation for the basename/dirname of the argument *(note the `{/}`)*

```fish
mkdir symbols private-symbols

fd --maxdepth 1 -t f \
    . ./System/Library/*Frameworks/*.framework/Versions/A/ \
    -x sh -c 'nm --demangle --defined-only --extern-only {} > symbols/{/}'
fd --maxdepth 1 -t f \
    . ./System/Library/*Frameworks/*.framework/Versions/A/ \
    -x sh -c 'nm --demangle --defined-only {} > private-symbols/{/}'

```

Searching for `clamshell` gives us interesting results. The most notable is this one inside SkyLight:

```fish
~/Temp/dyld ❯ rg -i clamshell

symbols/SkyLight
1710:00000001d44bce70 S _kSLSDisplayControlRequestClamshellState
```

`SkyLight.framework` is what handles window and display management in macOS and it usually exports enough symbols that we can use from Swift so I'm inclined to follow this path.

Let's see if the internet has anything for us. I usually search for code on [SourceGraph](https://sourcegraph.com) as it has indexed some large macOS repos with dyld dumps. Looking for  `RequestClamshellState` gives us [something far more interesting](https://sourcegraph.com/github.com/apple-oss-distributions/PowerManagement/-/blob/pmconfigd/PMDisplay.m?L527:6&subtree=true) though:

{{< img src="sourcegraph-requestclamshellstate.png" alt="searching RequestClamshellState on SourceGraph" sizes="(min-width: 60em) 810px, 90vw" >}}

---

Looks like Apple open sourced the power management code, nice! It even has recent ARM64 code in there, are we that lucky?

Here's an excerpt of something relevant to our cause:


```objc
SLSDisplayPowerControlClient *gSLPowerClient = nil;

enum {
    kPMClamshellOpen          = 1,
    kPMClamshellClosed        = 2,
    kPMClamshellUnknown       = 3,
    kPMClamshellDoesNotExist  = 4
};

void handleSkylightCheckIn(void)
{
// ...
    // create ws power control client
    NSError *err = nil;
    gSLPowerClient = [[SLSDisplayPowerControlClient alloc] initAsyncPowerControlClient:&err notifyQueue:_getPMMainQueue() notificationType:kSLDCNotificationTypeNone notificationBlock:^(void *dict) {
        if (dict != nil) {
            handleSkylightNotification(dict);
        } else {
            ERROR_LOG("Received a nil dictionary from WindowServer callback");
        }
    }];
// ...
}

void requestClamshellState(SLSClamshellState state)
{
    /* Forward clamshell state to WindowServer
     A) a request with a clamshell state of close in interpreted as a turn off clamshell display (clamshell close)
     B) a request with a clamshell state of open in interpreted as a turn on internal and ANY external displays (clamshell open)
     */

    if (!gSLCheckIn) {
        ERROR_LOG("WindowServer has not checked in. Refusing to change clamshell display state");
        return;
    }

    NSError *err = nil;
    NSMutableDictionary *request = [[NSMutableDictionary alloc] initWithCapacity:1];
    NSNumber *ns_state = [[NSNumber alloc] initWithUnsignedChar:state];
    [request setValue:ns_state forKey:kSLSDisplayControlRequestClamshellState];
    SLSDisplayControlRequestUUID uuid = [gSLPowerClient requestStateChange:(NSDictionary *const)request error:&err];
    if ([err code] != 0) {
       ERROR_LOG("Clamshell requestStateChange returned error %{public}@", err);
    } else {
       INFO_LOG("requestClamshellState: state %u, Received uuid %llu", state, uuid);
        struct request_entry *entry = (struct request_entry *)malloc(sizeof(struct request_entry));
        entry->uuid = uuid;
        entry->valid = true;
        STAILQ_INSERT_TAIL(&gRequestUUIDs, entry, entries);
    }
    if (request) {
       [ns_state release];
       [request release];
    }
    if (err) {
        [err release];
    }
}
```

So it's instantiating an `SLSDisplayPowerControlClient` then calling its `requestStateChange` method. `SLS` is a prefix related to SkyLight *(probably standing for SkyLightServer)*, let's see if we have that code in our version of the framework.

I prefer to do that using [Hopper](https://www.hopperapp.com/) and its **Read File From DYLD Cache** feature which can extract a framework from the currently in-use cache:

{{< img src="hopper-read-file-dyld-cache.png" alt="Hopper menu item showing Read file from DYLD cache" sizes="(min-width: 60em) 810px, 90vw" >}}

{{< img src="hopper-skylight-displaypowercontrol.png" alt="Hopper showing SLSDisplayPowerControlClient" sizes="(min-width: 60em) 500px, 90vw" maxWidth="500px" >}}

Ok the class and methods are there, let's look for what uses them. Since it's most likely a daemon handling power management, I'll look for it in `/System/Library`.

And looks like `powerd` is what we're looking for, containing exactly the code that we saw on SourceGraph.

```fish
❯ rg -uuu requestClamshellState /System/Library/ 2>/dev/null

/System/Library/CoreServices/powerd.bundle/powerd: binary file matches (found "\0" byte around offset 4)

❯ hopperv4 -e /System/Library/CoreServices/powerd.bundle/powerd
```


{{< img src="hopper-requeststatechange.png" alt="Hopper pseudocode calling requestStateChange" sizes="(min-width: 60em) 810px, 90vw" >}}

## Writing the code
To link and use `SLSDisplayPowerControlClient` we need some headers, as Swift doesn't have the method signatures available.

Looking for `SLSDisplayPowerControlClient` on SourceGraph gives us [more than we need](https://sourcegraph.com/github.com/cmsj/ApplePrivateHeaders/-/blob/macOS/11.3/System/Library/PrivateFrameworks/SkyLight.framework/Versions/A/SkyLight/SLSDisplayPowerControlClient.h?L10:26&subtree=true).

Let's create a bridging header so that Swift can link to Objective-C symbols, and a Swift file to where we'll try to replicate what `powerd` does.


```sh
mkdir clamshell && cd clamshell
touch Bridging-Header.h Clamshell.swift
```

##### Bridging-Header.h

```objc
#import <Foundation/Foundation.h>

@interface SLSDisplayPowerControlClient {}

- (id)initAsyncPowerControlClient:(id*)arg1 notifyQueue:(id)arg2 notificationType:(UInt8)arg3 notificationBlock:(void (^)(NSDictionary*))notificationBlock;
- (id)initPowerControlClient:(id*)arg1 notifyQueue:(id)arg2 notificationType:(UInt8)arg3 notificationBlock:(void (^)(NSDictionary*))notificationBlock;

- (unsigned long long)requestStateChange:(id)arg1 error:(id*)arg2;
@end

extern NSString* kSLSDisplayControlRequestClamshellState;
UInt8 kSLDCNotificationTypeNone = 0;
```

##### Clamshell.swift

```swift
import Foundation

enum ClamshellState: Int {
    case open = 1
    case closed = 2
    case unknown = 3
    case doesNotExist = 4
}

var err: AnyObject?
let skyLightPowerClient = SLSDisplayPowerControlClient(powerControlClient: &err, notifyQueue: DispatchQueue.main, notificationType: kSLDCNotificationTypeNone) { dict in
    print(dict as Any)
}

func requestClamshellState(_ state: ClamshellState) {
    // Send the request
    let request: [AnyHashable: Any] = [
        kSLSDisplayControlRequestClamshellState: NSNumber(value: state.rawValue)
    ]

    var err: AnyObject?
    let uuid = skyLightPowerClient!.requestStateChange(request, error: &err)

    // Check the response
    if (err as! NSError?)?.code != 0 {
        print("Clamshell requestStateChange returned error", err?.localizedDescription ?? "")
    } else {
        print("requestClamshellState: state %u, Received uuid %llu", state, uuid)
    }
}

print(skyLightPowerClient!)
requestClamshellState(.closed)
```

---

### Compiling...

To compile the binary using `swiftc` we have to point it to the location of SkyLight.framework which is located at `/System/Library/PrivateFrameworks`.

We then tell it to link the framework using `-framework SkyLight` and import our bridging header. Then we run the resulting binary.

*I prefer to run this using `entr` to watch the files for changes. With the code editor on the left and the terminal on the right, I can iterate and try things faster by just editing and saving the file, then watch the output on the right.*

```sh
swiftc \
    -F/System/Library/PrivateFrameworks \
    -framework SkyLight \
    -import-objc-header Bridging-Header.h \
    Clamshell.swift -o Clamshell
./Clamshell

# For faster iteration, watch file changes with entr:

echo Clamshell.swift Bridging-Header.h | entr -rs '\
	swiftc -F/System/Library/PrivateFrameworks \
		-framework SkyLight \
		-import-objc-header Bridging-Header.h \
		 Clamshell.swift -o Clamshell \
	&& ./Clamshell'
```

Well.. it's not working. The error is not helpful at all, there's nothing on the internet related to it.

```swift
<SLSDisplayPowerControlClient: 0x600000fb8720>
Clamshell requestStateChange returned error The operation couldn’t be completed. (CoreGraphicsErrorDomain error 1004.)
```

### Looking for errors
Maybe the system log has something for us. One can check that using Console.app but I prefer looking at it in the Terminal through the `/usr/bin/log` utility.

```sh
log stream --predicate 'eventMessage contains "Clamshell"'
```

Something from AMFI about the binary signature. CMS stands for **[Cryptographic Message Syntax](https://medium.com/csit-tech-blog/demystifying-ios-code-signature-309d52c2ff1d)** which is what `codesign` adds to a binary when it signs it with a certificate.


```swift
kernel: (AppleMobileFileIntegrity) AMFI: '/Users/alin/Temp/dyld/clamshell/Clamshell' has no CMS blob?
kernel: (AppleMobileFileIntegrity) AMFI: '/Users/alin/Temp/dyld/clamshell/Clamshell': Unrecoverable CT signature issue, bailing out.
tccd: [com.apple.TCC:access] AUTHREQ_ATTRIBUTION: msgID=60667.1, attribution={responsible={TCCDProcess: identifier=kitty, pid=19959, auid=501, euid=501, responsible_path=/Applications/kitty.app/Contents/MacOS/kitty, binary_path=/Applications/kitty.app/Contents/MacOS/kitty}, requesting={TCCDProcess: identifier=Clamshell, pid=60667, auid=501, euid=501, binary_path=/Users/alin/Temp/dyld/clamshell/Clamshell}, },
```

I have GateKeeper disabled and running the binary from a terminal that's added to the special Developer Tools section of **Security & Privacy**, so this shouldn't cause any problems.

I checked just to be sure, and signing it with my $100/year Apple Developer certificate gets rid of the `CMS blob` error but doesn't change anything in the result.

---

{{< rawhtml >}}
<div class="flex items-center justify-between flex-column flex-row-ns">
    <div class="mw6">
        <h3>Phew, let's take a break</h3>
        <p class="mt4 f5">I just arrived <small><i>after a long train ride</i></small> at the house I'm rebuilding with my wife, and wanted to share this nice view with you 😌</p>
        <p class="f5">It's January, but the sun is warming our faces and the hazelnut trees are already producing their <a href="https://gardenplannerwebsites.azureedge.net/blog/catkins-2x.jpg">yellow catkins</a>.</p>
        <p class="mt4 f5">Ten years ago, the children of the house's previous owners were walking in knee deep snow and coasting downhill on their wooden sleds, hurting a few young fir trees on the way down. 🌲</p>
        <p class="f5 mb4">Seasons are changing.</p>
    </div>
    <img class="ma0" src="/images/breaza-sun-in-january.jpg" alt="breaza sun in January" height=400>
</div>
{{< /rawhtml >}}

---

## Digging deeper

Some system capabilities can only be accessed if the binary has been signed by Apple and has specific [entitlements](https://secret.club/2020/08/14/macos-entitlements.html). Checking for `powerd`'s entitlements gives us something worrying.

The binary seems to use `com.apple.private.*` entitlements. This usually means that some APIs will fail if the required entitlements are not present.

```fish
> codesign -d --entitlements - /System/Library/CoreServices/powerd.bundle/powerd

Executable=/System/Library/CoreServices/powerd.bundle/powerd
[Dict]
	...
	[Key] com.apple.private.SkyLight.displaypowercontrol
	[Value]
		[Bool] true
	...
```

We can try to add the entitlements ourselves. We just need to create a plist file and use it in `codesign`:

##### Entitlements.plist
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>com.apple.private.SkyLight.displaypowercontrol</key>
        <true/>
    </dict>
</plist>
```

Sign the binary with entitlements and run it:
```sh
❯ codesign -fs $CODESIGN_CERT --entitlements Entitlements.plist Clamshell
❯ ./Clamshell
Job 1, './Clamshell' terminated by signal SIGKILL (Forced quit)
```

Looks like we're getting killed instantly. The log stream shows AMFI is doing that because we're not Apple and we're not supposed to use that entitlement.

```swift
kernel: mac_vnode_check_signature: /Users/alin/Temp/dyld/clamshell/Clamshell: code signature validation failed fatally: When validating /Users/alin/Temp/dyld/clamshell/Clamshell:
  Code has restricted entitlements, but the validation of its code signature failed.
Unsatisfied Entitlements: com.apple.private.SkyLight.displaypowercontrol

kernel: (AppleSystemPolicy) ASP: Security policy would not allow process: 57234, /Users/alin/Temp/dyld/clamshell/Clamshell
amfid: /Users/alin/Temp/dyld/clamshell/Clamshell not valid: Error Domain=AppleMobileFileIntegrityError Code=-413 "No matching profile found" UserInfo={NSURL=file:///Users/alin/Temp/dyld/clamshell/Clamshell, unsatisfiedEntitlements=<CFArray 0x155e1b600 [0x1f0e613a8]>{type = immutable, count = 1, values = (
    0 : <CFString 0x155e12db0 [0x1f0e613a8]>{contents = "com.apple.private.SkyLight.displaypowercontrol"}
)}, NSLocalizedDescription=No matching profile found}
```

### AMFI
What's this AMFI exactly and why is it telling us what we can and cannot do on our own device?

The acronym stands for [Apple Mobile File Integrity](https://eclecticlight.co/2018/12/29/amfi-checking-file-integrity-on-your-mac/) and it's the process enforcing code signature at the system level.

By default, the OS locks these private APIs because if we would be able to use them, a malware or a bad actor would be able to do it as well. With it locked by default, malware authors are deterred from trying to use these APIs on targets of lower importance as this would usually need a 0-day exploit.

In the end it's just another layer of security, and if in the rare case someone needs to bypass it, Apple provides a way to do it. The process involves disabling **System Integrity Protection** and adding `amfi_get_out_of_my_way=1` as a boot arg.

```fish
# Inside a Recovery terminal (to disable SIP)

> csrutil disable
> reboot

# Inside a normal terminal after disabling SIP

> sudo nvram boot-args="amfi_get_out_of_my_way=1"
> sudo reboot now
```

I don't recommend doing this as it puts you at great risk, since the system volume is no longer read only, and code signatures are no longer enforced.

I only keep this state for research that I do in short periods of time, then turn SIP back on for normal day to day usage.

In case you need to revert the above changes:

```fish
# Inside a normal terminal before enabling SIP

> sudo nvram boot-args=""

# Inside a Recovery terminal (to enable SIP)

> csrutil enable
> reboot
```

---

### No more AMFI?

Unfortunately even after disabling AMFI, we're still encountering the `CoreGraphicsError 1004`. It's true, AMFI is not complaining about the entitlements anymore, they're accepted and the binary is not `SIGKILL`ed.

But we still can't get into clamshell mode using just software.

## [Frida](https://frida.re/)
If you haven't heard of it, Frida is this awesome tool that lets you inject code into already running processes, hook functions by name *(or even by address)*, observe how and when they're called, check their arguments and even make your own calls.

Let me share with you another macOS boot arg that I like:

```fish
sudo nvram boot-args=-arm64e_preview_abi
```

This one enables code injection. Now we can use Frida to hook the SkyLight power control methods to see how they are called as we close and open the lid:

```fish
> sudo frida-trace -t SkyLight -m '-[SLSDisplayPowerControlClient *]' powerd

// Closing the lid
           /* TID 0x5427 */
  4617 ms  SLSDisplayControlRequestClamshellStateKey: 2
  4617 ms  -[SLSDisplayPowerControlClient requestStateChange:0x13cf06a60 error:0x16b8ca828]
  4628 ms     | -[SLSDisplayPowerControlClient service]
  4628 ms     | -[SLSDisplayPowerControlClient sendStateChangeRequest:0x13cf06a60 uuid:0x16b8ca7e0]
  4628 ms     |    | -[SLSDisplayPowerControlClient service]

// Opening the lid
           /* TID 0x8a17 */
 10537 ms  SLSDisplayControlRequestClamshellStateKey: 1
 10537 ms  -[SLSDisplayPowerControlClient requestStateChange:0x13cc1e1c0 error:0x16b9567a8]
 10538 ms     | -[SLSDisplayPowerControlClient service]
 10538 ms     | -[SLSDisplayPowerControlClient sendStateChangeRequest:0x13cc1e1c0 uuid:0x16b956760]
 10538 ms     |    | -[SLSDisplayPowerControlClient service]
```

We got our confirmation at least. `powerd` is indeed calling `SLSDisplayPowerControlClient.requestStateChange(2)` when closing the lid.

Let's check what happens when we try to call that method in `Clamshell.swift`.

We first add the line `readLine(strippingNewline: true)` at the top of the `Clamshell.swift` file to make the binary wait for us to press `Enter`. This is so that we have a running process that we can attach to with Frida.

```fish
> sudo frida-trace -t SkyLight -m '-[SLSDisplayPowerControlClient *]' Clamshell

           /* TID 0x103 */
  1475 ms  SLSDisplayControlRequestClamshellStateKey: 2
  1475 ms  -[SLSDisplayPowerControlClient requestStateChange:0x600001d64510 error:0x16d8d3c90]
  1479 ms     | -[SLSDisplayPowerControlClient service]
  1479 ms     | -[SLSDisplayPowerControlClient sendStateChangeRequest:0x600001d64510 uuid:0x16d8d3a10]
  1479 ms     |    | -[SLSDisplayPowerControlClient service]
```

Everything looks the same, seems that we're not looking deep enough.

The request method seems to access the `service` property which is an `SLSXPCService`. [XPC Services](https://developer.apple.com/documentation/xpc) are what macOS uses for low-level interprocess communication.

A process can expose an XPC Service using a label (e.g. `com.myapp.RemoteControlService`) and listen to requests coming through, other processes can connect to it using the same label and send requests.

The system handles the routing part. And the authentication part.

Looks like an XPC Service can also be [restricted to specific code signing requirements](https://developer.apple.com/documentation/foundation/nsxpclistener/3943310-setconnectioncodesigningrequirem), is it possible that this is what we're running into here?

Let's trace `SLSXPCService` methods as well using Frida:

```fish
> sudo frida-trace -t SkyLight -m '-[SLSDisplayPowerControlClient *]' -m '-[SLSXPCService *]' powerd

// Closing the lid while observing powerd
           /* TID 0x518b */
  3029 ms  -[SLSDisplayPowerControlClient requestStateChange:0x139621c60 error:0x16f0c2828]
  3029 ms  SLSDisplayControlRequestClamshellStateKey: 2
  3043 ms     | -[SLSDisplayPowerControlClient service]
  3043 ms     | -[SLSDisplayPowerControlClient sendStateChangeRequest:0x139621c60 uuid:0x16f0c27e0]
  3043 ms     |    | -[SLSDisplayPowerControlClient service]
  3043 ms     |    | -[SLSXPCService sendXPCDictionary:0x13a913be0]
  3043 ms     |    |    | -[SLSXPCService reinitConnection]
  3043 ms     |    |    |    | -[SLSXPCService enabled]
  3043 ms     |    |    |    | -[SLSXPCService enabled]
  3043 ms     |    |    |    | -[SLSXPCService connected]
  3043 ms     |    |    | -[SLSXPCService connection]
  3452 ms  -[SLSXPCService handleXPCEvent:0x13ad0ea40]
  3452 ms     | -[SLSXPCService enabled]
  3452 ms     | -[SLSXPCService cfStringToCStringPtr:0x1f3133020]
  3452 ms     | -[SLSXPCService connected]

> sudo frida-trace -t SkyLight -m '-[SLSDisplayPowerControlClient *]' -m '-[SLSXPCService *]' Clamshell

// Trying to send the clamshell request in software
  1435 ms  -[SLSDisplayPowerControlClient requestStateChange:0x6000014d4030 error:0x16b123c90]
  1435 ms  SLSDisplayControlRequestClamshellStateKey: 2
  1444 ms     | -[SLSDisplayPowerControlClient service]
  1444 ms     | -[SLSDisplayPowerControlClient sendStateChangeRequest:0x6000014d4030 uuid:0x16b123a10]
  1444 ms     |    | -[SLSDisplayPowerControlClient service]
  1444 ms     |    | -[SLSXPCService sendXPCDictionary:0x600003ec4000]
  1444 ms     |    |    | -[SLSXPCService reinitConnection]
  1444 ms     |    |    |    | -[SLSXPCService enabled]
  1444 ms     |    |    |    | -[SLSXPCService connected]
  1444 ms     |    |    |    | -[SLSXPCService autoreconnect]
  1444 ms     |    |    |    | -[SLSXPCService enabled]
Process terminated

// ...we're missing this stuff
//  3043 ms     |    |    | -[SLSXPCService connection]
//  3452 ms  -[SLSXPCService handleXPCEvent:0x13ad0ea40]
//  3452 ms     | -[SLSXPCService enabled]
//  3452 ms     | -[SLSXPCService cfStringToCStringPtr:0x1f3133020]
//  3452 ms     | -[SLSXPCService connected]
```

Great! or not?

I'm not sure if I should be happy that we found that our clamshell request doesn't work because we don't have an XPC connection, or if I should be worried that this means we won't be able to make this work with SIP enabled.

I guess it's time to go deeper to find out.

## XPC Services

Now that we have access to Frida, we can use the handy [xpcspy](https://github.com/hot3eed/xpcspy) tool to sniff the XPC communication of `powerd`.

I'm thinking maybe we can find the endpoint name of the XPC listener and just connect to it and send a raw message directly, instead of relying on SkyLight to do that.

```fish
> sudo xpcspy --parse powerd

// Closing the lid

xpc_connection_send_message
<OS_xpc_connection: <connection: 0x13a808820> { name = (anonymous), listener = false, pid = 30630, euid = 88, egid = 88, asid = 100014 }>
<OS_xpc_dictionary> { count = 4 contents =
    "PayloadType" => <OS_xpc_uint64: <uint64: 0x81917509705f5717>: 3>
    "Command" => <OS_xpc_uint64: <uint64: 0x81917509705f572f>: 4>
    "Payload" => <data> { length = 92 bytes, contents = {
	    SLSDisplayControlRequestClamshellStateKey = 2;
	}
    "UUID" => <OS_xpc_uint64: <uint64: 0x81917509705f5647>: 41>

}
```

So we have `name = (anonymous), listener = false, pid = 30630`.

An anonymous listener, can it get even worse? The PID coincides with `WindowServer --daemon` so it's definitely the message we're also trying to send.  But with an anonymous listener, we're stuck to relying on SkyLight's exported code to reach it.

I guess we need to go back to do some old-school assembly reading.

---

### Filling empty spaces

After renaming some sub-procedures in Hopper, looking at the graph reveals the different code paths that `powerd` and `Clamshell` are taking through `SLSXPCService.reinitConnection`.

#### powerd
1. sees that the service's `enabled` and `connected` properties are `true`
2. so it gets out of `reinitConnection`
3. and straight into sending the XPC dictionary through the available `connection`.

#### Clamshell
- sees that `enabled`, `connected` and `autoreconnect` are `false`
	- so it fails with a `CGError`
- if those properties were `true` it would go on the right-side code path which
	- checks if properties at `0x20` and `0x28` are non-zero
	- then proceeds to reconnect.

{{< img src="hopper-trace-reinitconnection.png" alt="Hopper graph showing reinitConnection" sizes="(min-width: 60em) 810px, 90vw" >}}

Adding some `Memory.readPointer` calls inside `__handlers__/SLSXPCService/reinitConnection.js` shows us what SkyLight is expecting to see at `0x20` and `0x28`:

Two `NSMallocBlock`s right after the `OS_xpc_connection` and the `OS_dispatch_queue_serial` properties.

```swift
  5502 ms   -[SLSXPCService reinitConnection]
  5502 ms       arg0 obj: <SLSXPCService: 0x11f50f130>

  5502 ms       Memory.readPointer 0x8 0x101

  5502 ms       Memory.readPointer 0x10 0x11f50bb10
  5502 ms       SLSXPCService at 0x10 <OS_xpc_connection: <connection: 0x11f50bb10> { name = (anonymous), listener = false, pid = 396, euid = 88, egid = 88, asid = 100014 }>

  5502 ms       Memory.readPointer 0x18 0x11df05970
  5502 ms       SLSXPCService at 0x18 <OS_dispatch_queue_serial: Power Management main queue>

  5502 ms       Memory.readPointer 0x20 0x11f50a740
  5502 ms       SLSXPCService at 0x20 <__NSMallocBlock__: 0x11f50a740>

  5502 ms       Memory.readPointer 0x28 0x11f50a770
  5502 ms       SLSXPCService at 0x28 <__NSMallocBlock__: 0x11f50a770>
```

Judging by the contents of [SLSXPCService.h](https://sourcegraph.com/github.com/cmsj/ApplePrivateHeaders/-/blob/macOS/11.3/System/Library/PrivateFrameworks/SkyLight.framework/Versions/A/SkyLight/SLSXPCService.h?L22-23), those are the closures for `errorBlock` and `notificationBlock`:

```objc
@interface SLSXPCService : NSObject <SLSXPCServiceProtocol> {
	char _enabled;
	char _connected;
	char _setTarget;
	char _autoreconnect;
	NSObject*<OS_xpc_object> _connection;
	NSObject*<OS_dispatch_queue> _notifyQueue;

	/* This would be 0x20 */ id _errorBlock;
	/* This would be 0x28 */ id _notificationBlock;

	/*^block*/id _clientErrorBlock;
	/*^block*/id _clientNotificationBlock;
}
```

I'm inching closer to the good code path but I seem to never get there.

So here's what I did so far in `Clamshell.swift` before calling `requestClamshellState`:

```swift
guard let service = skyLightPowerClient.service else {
    print("SLSXPCService is nil")
    exit(1)
}

service.autoreconnect = true
service.errorBlock = { err in
    print("service.errorBlock", err)
}
service.notificationBlock = { notification in
    print("service.notificationBlock", notification)
}
```

After calling `requestClamshellState`, the code crashes with `SIGSEGV` inside `createNoSenderRecvPairWithQueue:errorHandler:eventHandler:` because it branches to the `0x0` address.

```asm
// The crashing instruction is:
ldr    x8, [x19, #0x28]

// Because the memory at [x19, #0x28] contains 0x0

// And register x19 contains:
(lldb) po $x19
<__NSMallocBlock__: 0x600000c08330>
 signature: "v8@?0"
 invoke   : 0x1a081a958 (/System/Library/PrivateFrameworks/SkyLight.framework/Versions/A/SkyLight`__75-[SLSXPCService createNoSenderRecvPairWithQueue:errorHandler:eventHandler:]_block_invoke)
 copy     : 0x1a05e0a18 (/System/Library/PrivateFrameworks/SkyLight.framework/Versions/A/SkyLight`__copy_helper_block_e8_32b40r)
 dispose  : 0x1a05e09d4 (/System/Library/PrivateFrameworks/SkyLight.framework/Versions/A/SkyLight`__destroy_helper_block_e8_32b40r)

```

### Giving up (for now)

Unfortunately I'm a bit lost here. I'll take a break and hope that the solution comes in a dream or on a long walk like in those mythical stories.

The article is already longer than I'd be inclined to read so if anyone reaches this point, congrats, you have the patience of a monk.

If there are better ways to approach a problem like this one, I'd be glad to hear about it through [the contact form](/contact).

I'm not always happy to learn that I've wasted 4 days on a problem that could have been solved in a few hours with the right tools, but at least I'll learn how not to bore people with writings on rudimentary tasks next time.
