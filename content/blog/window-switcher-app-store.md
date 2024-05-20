---
title: "A window switcher on the Mac App Store? Is it even possible?"
subtitle: "Focusing a specific window on macOS felt too cumbersome. I tried revamping that from inside the confines of an App Store app. So is it possible?"
description: "Focusing a specific window on macOS felt too cumbersome. I tried revamping that from inside the confines of an App Store app. So is it possible?"
excerpt: |-
    Have you ever tried Command Tabbing through 10 different apps, 50 times a day? What a pleasant experience, right?

    Well it definitely wasn't for me, so I created an app switcher, published it on the App Store, then noticed I also needed a window switcher, noticed there's no such thing on the App Store and proceeded to find a way to create one.
date: 2022-08-02T21:01:04+03:00
author: Alin Panaitiu
draft: false
image: window-switcher-app-store.jpg
images:
-  https://img.panaitiu.com/_/og/plain/https%3A%2F%2Falinpanaitiu.com%2Fimages%2Fwindow-switcher-app-store.jpg@webp
tags:
    - macbook
    - macos app
    - rcmd
    - app switch
    - command tab
    - window switch
    - menu bar
    - app store
    - sandbox
series:
    - Mac App Store
categories:
    - macOS apps
pageStyles:
  - file: article.sass
    media: "(prefers-color-scheme: light), (prefers-color-scheme: no-preference)"
  - file: article-dark.sass
    media: "(prefers-color-scheme: dark)"

---

> *Not really, no. Not without annoying workarounds and a confusing user experience.*

Another email, another annoyed user: *Firefox not loading websites when launched through rcmd! It works when launched from Alfred.. Please fix ASAP!!* I’m gonna fix this Firefox issue once and for all!

Launch Xcode, open the **![rcmd icon](/images/icons/rcmd_16@2x.webp#x16) [rcmd](https://lowtechguys.com/rcmd)** project, check the `launchApp` function code, it’s just a `NSWorkspace.open` call on Firefox.app, what does Alfred do differently?

Disassemble Alfred.app in Hopper, look for `NSWorkspace.open`, of course it’s there, it’s the exact same thing.

{{< img src="alfred-hopper.png" alt="screenshot of hopper showing where open is used in Alfred code" sizes="(min-width: 60em) 810px, 90vw" >}}

Try `open /Applications/Firefox.app` in a terminal, it works, websites load as expected.

Breakpoint on `launchApp`, check the debugger again, let’s be rigorous, what am I really calling `open` on?

Argument is `/System/Volumes/Data/Applications/Firefox.app` which is just a symlink to `/Applications/Firefox.app` right? .. or was it the other way around? Anyway let’s just try it for the sake of it, I’m desperate.

Run `open /System/Volumes/Data/Applications/Firefox.app`, huh?? no websites load? THAT WAS IT?!

Add `path.replacingOccurrences(of: "/System/Volumes/Data", with: "")`, build, run, hold `Right Command`, press `F`, Firefox launches and holy cow everything works!!

I don’t even care why anymore, let’s just release this fix on the App Store.

And while I’m at it, why not try to add that window switching capability that people have been asking about?

I remember something about Accessibility permissions not being available in the sandbox, but I just used an App Store app that was able to request the permissions so there has to be a way, how hard could it be?

***Well it turns out it’s pretty darn hard, and I’m still working on this window switching thing to this day.. sigh.. let me tell you about it.***

---

## Apps vs windows
There’s an important distinction between switching windows and switching apps on the Mac. As opposed to Microsoft Windows where you just `Alt-Tab` through .. well, **windows**, on macOS you `Command Tab` through **apps** by default. When an app with multiple windows is focused, `Command backtick` will cycle through the windows of that app.

{{< img src="keyboard-cmd-tab-backtick.png" alt="keyboard with command tab and backtick keys highlighted" sizes="(min-width: 60em) 810px, 90vw" >}}

Six years ago I was a Windows power user, and when I got my first Mac, Command Tabbing through apps felt very weird. Suddenly I was closing all windows of **![sublime text icon](/images/icons/sublime_16@2x.webp#x16) Sublime** but its icon was still there in the Command Tab list, or I would minimize **![chrome icon](/images/icons/chrome_16@2x.webp#x16) Chrome** and focusing its icon didn’t unminimize it. The app vs window distinction just didn’t exist in my mind.

Now, after 6 years, the macOS way feels a lot more intuitive:
- I mostly switch between apps with a single window (browser, terminal etc.)
- There’s a very small subset of apps where I might have more than one window (code editor, image/PDF viewer)
- I might want to keep my code editor app running even after I closed all its windows, so I can have it load instantly when opening a new window to edit a file/project
- Minimizing windows that are irrelevant at the moment allows me to cycle through the relevant ones with `Command backtick`
- No need to see a thumbnail of each window, when almost all apps are single window

Of course it might just be the power of habit, after all I was able to be just as productive with the Windows way in the past ¯\\\_(ツ)\_/¯

## Command Tab Tab Tab Tab…

The app centric approach is nice but having to switch between 10 different apps at a time gets annoying fast.

{{< rawhtml >}}
<div style="width: 100%; display: flex; justify-content: center; align-items: center;">
    <video autoplay loop muted playsinline preload="none" style="width: 100%; max-width: 400px; margin: 10px auto; border-radius: 8px;" poster="/images/cmdtab-frustrating-fast.webp" src="/video/cmdtab-frustrating-fast.mp4">
    </video>
</div>
{{< /rawhtml >}}

*Pressing Tab 5 times in a row to get to the app I want could be categorized as a first world problem and I should just get used to it.  But doing that 50 times a day and having to always visually check if I chose the right icon, tends to break my flow of thinking, and makes me get tired faster because of all the context switching.*

That’s the main reason I created **![rcmd icon](/images/icons/rcmd_16@2x.webp#x16) [rcmd](https://lowtechguys.com/rcmd)**, to switch apps without thinking about switching apps.

My right thumb rests nicely on the `Right Command` key and I barely use that easy to reach key. So I turned it into a dedicated app switching key.

### Dynamic assignments

I decided to dynamically assign each app the first letter of its name so that I don’t have to try to remember *what key did I assign to Xcode?*. I just hold `Right Command` and press `X` without any mental effort because I know I have no other app starting with `X`.

And if I forgot that **![Xcode icon](/images/icons/xcode_16@2x.webp#x16) Xcode** is not already running *(or if it crashes in the background like it sometimes does)*, rcmd launches it automatically *(since I clearly wanted it running if I tried to focus it)*.

### Static assignments

Xcode is a happy case though. I have so many apps starting with `S` that I decided custom assignments might be a better fit for that. I left Sublime Text for the `S` key since it’s my most used app, and then assigned mnemonic keys for others:
- `O` for ![soulver app icon](/images/icons/soulver_16@2x.webp#x16) S**o**ulver
- `P` for ![spotify app icon](/images/icons/spotify_16@2x.webp#x16) S**p**otify
- `E` for ![sketch app icon](/images/icons/sketch_16@2x.webp#x16) Sk**e**tch *(because `K` is taken by the ![kitty app icon](/images/icons/kitty_16@2x.webp#x16) Kitty terminal)*
- `B` for ![safari app icon](/images/icons/safari_16@2x.webp#x16) Safari **b**rowser
- Other rarely used apps *(SF Symbols, Slack, Sublime Merge)* will be reachable by cycling using `rcmd-rshift-s` *(it’s good enough for me as I rarely have those open)*

### Seek and hide

Often I need to check the status of an app briefly and then get back to what I was doing. Some examples
- ![kitty app icon](/images/icons/kitty_16@2x.webp#x16)  check a long running task in the terminal
- ![mail app icon](/images/icons/mail_16@2x.webp#x16)  check if I got an email I’m waiting for while notifications are paused
- ![spotify app icon](/images/icons/spotify_16@2x.webp#x16)  see what’s this dope song that started playing from my Discover Weekly playlist

That’s why I added the **Hide** action in rcmd.

Now I just hold `Right Command` and press `K` to check the **![kitty app icon](/images/icons/kitty_16@2x.webp#x16) [Kitty](https://sw.kovidgoyal.net/kitty/)** terminal, then, without lifting any finger, press `K` again to hide it and get back to what I was doing.

*This also allows the system to activate **App Nap** for the hidden app and put it into a lower energy usage state until I need it again.*

{{< rawhtml >}}
<div style="width: 100%; display: flex; justify-content: center; align-items: center; flex-direction: column;">
    <video autoplay loop muted playsinline preload="none" style="width: 100%; max-width: 782px; margin: 10px auto; border-radius: 12px;" poster="/images/rcmd-demo-macbook.webp" src="/video/rcmd-demo-macbook.mp4">
    </video>
    <em>Using rcmd on my MacBook Pro 14"</em>
</div>
{{< /rawhtml >}}


---

## Is window switching even needed?

Unfortunately yes, there are many cases where an app might have a lot of windows open:
- ![sublime text icon](/images/icons/sublime_16@2x.webp#x16)  Separate projects/folders open in Sublime Text
- ![pages icon](/images/icons/pages_16@2x.webp#x16)  Multiple documents in Pages or Microsoft Word
- ![preview icon](/images/icons/preview_16@2x.webp#x16)  Lots of PDFs open for referencing in Preview

### Available solutions
1. **App Expose:** Command Tab allows pressing the `↓ Down Arrow` key with the app icon selected, to expose all the windows of that app for visual selection.
    - It’s nice and useful for when you churn windows a lot, but it’s way too slow for cases when you mostly have the same windows open.
2. **Command backtick `` ` ``:** this native macOS hotkey will cycle through the windows of the current app but we’re back to square one where you have to visually analyze each window to see if you got the right one in focus.

3. **[Alt-Tab](https://alt-tab-macos.netlify.app/):** this is a really nice open source app which replicates the Microsoft Windows way of selecting windows by thumbnails.
    - It’s what I used for a long time, until I got too frustrated with the fact that all my seven Sublime Text windows look exactly the same and I have to also read the whole window title to find the one I want to focus.
4. [**Contexts.co**](https://contexts.co): a fuzzy searcher for window titles. I’ve used it in the past and it was definitely faster than the rest but it still required more key presses than I wanted
    - I don’t really need to search the whole window title, just the project name.
5. **Stage Manager:** the new addition in macOS Ventura, which in its current state is just *discoverable Spaces*.
    - That’s the feeling I got from using it: stages are just like spaces, but more visible (through the left sidebar) and easier to reach for (by clicking on them or by focusing a window in a specific stage).
    - It still doesn’t provide any keyboard control and moving specific windows in and out of the stages requires too much work with the mouse.
    - At least for Spaces I had [yabai](https://github.com/koekeishiya/yabai) to provide keyboard shortcuts for moving the current window to whatever space I wanted to.

### My preferred solution: the Right Option key

It’s a sunny day in Brașov, I’m on my balcony taking in the sun, testing and perfecting [XDR Brightness](https://lunar.fyi/#xdr) to make working in direct sunlight easier on my MacBook 14” while also rewriting parts of the **![Lunar Icon](/images/icons/lunar_16@2x.webp#x16) [Lunar](https://lunar.fyi)** UI in SwiftUI.

{{< rawhtml >}}
<div style="width: 100%; display: flex; justify-content: center; align-items: center; flex-direction: column;">
    <video autoplay loop muted playsinline preload="none" style="height: 80%; max-height: 250px; margin: 10px auto; border-radius: 8px;" poster="/images/auto-xdr-sunlight-brasov.webp" src="/video/auto-xdr-sunlight-brasov.mp4">
    </video>
    <em>Testing Auto XDR</em>
</div>
{{< /rawhtml >}}

I’ve already written a lot of SwiftUI boilerplate in my other projects, so I’m mostly copy pasting stuff between Sublime Text windows. I also have three Sublime windows with disassembled macOS private frameworks to look for the hidden functions I need to improve the XDR Brightness curve and responsiveness.

Juggling with all these windows suddenly became very frustrating.

Why can’t I focus exactly the window I want with one hotkey just like I focus apps with rcmd?

I’m probably going to have the same set of windows for the next few days, I know the names of the projects I have open in them, I could use the first letter of the project name to reference a specific window.

The `Right Command` key is taken, but right beside it stands another rarely used key: the `Right Option` key *(`ralt` for short)*

I want to be able to press `ralt-r` to focus the Sublime window containing the **r**cmd project, `ralt-l` to focus the **L**unar project, `ralt-v` for the **V**olum project, `ralt-p` to get to the **P**rivateFrameworks folder and so on.

The plan seems simple enough:
- get the list of windows and their title from the current app
- extract the first letter of the project name
- assign `Right Option` + `letter` to some `focusWindow` function
- get back to the real work

## Oh right … the sandbox

It’s not like the above hasn’t been done before, there are plenty of window switcher and snap/resize examples on macOS, some of them are even open source:
- [yabai](https://github.com/koekeishiya/yabai)
- [Amethyst](https://ianyh.com/amethyst/)
- [Rectangle](https://github.com/rxhanson/Rectangle)

One window snapping tools is even on the App Store: [Magnet](https://apps.apple.com/ro/app/magnet/id441258766?mt=12)

But why are there no window switchers on the App Store?

Well, for app switching, Apple provides a really nice API to enumerate and activate running apps without needing any intrusive permissions: [NSRunningApplication](https://developer.apple.com/documentation/appkit/nsrunningapplication)

*Finding Xcode and focusing it*
```swift
let apps = NSWorkspace.shared.runningApplications
let xcode = apps.first { app in
    app.bundleIdentifier == "com.apple.dt.Xcode"
}
xcode?.activate()
```

But there’s no such thing for enumerating the windows of those running apps. All of the apps that work with app windows, need to tap into the Accessibility API, the one that gives you full access to extract and modify the contents of everything visible and invisible.

{{< img src="yabai-accessibility-permissions.jpg" alt="system dialog with yabai requesting Accessibility permissions" sizes="(min-width: 60em) 810px, 90vw" >}}

And so, window enumeration becomes possible, by fetching the array of UI elements under the `AXWindows` attribute of an app.

But since a window is like just any other UI element, then there’s no `focus` or `activate` method, so how do these apps manage to focus a window?

Take a look at this nice and *intuitive* snippet extracted from [yabai](https://github.com/koekeishiya/yabai/blob/master/src/window_manager.c#L815):

```c
static void window_manager_make_key_window(ProcessSerialNumber *window_psn, uint32_t window_id)
{
    uint8_t bytes1[0xf8] = { [0x04] = 0xf8, [0x08] = 0x01, [0x3a] = 0x10 };
    uint8_t bytes2[0xf8] = { [0x04] = 0xf8, [0x08] = 0x02, [0x3a] = 0x10 };

    memcpy(bytes1 + 0x3c, &window_id, sizeof(uint32_t));
    memset(bytes1 + 0x20, 0xFF, 0x10);

    memcpy(bytes2 + 0x3c, &window_id, sizeof(uint32_t));
    memset(bytes2 + 0x20, 0xFF, 0x10);

    SLPSPostEventRecordTo(window_psn, bytes1);
    SLPSPostEventRecordTo(window_psn, bytes2);
}
```

Even though I knew that ***key window***  meant focused window in macOS terminology, it still took me a while to land on this code and start believing that this is really focusing a window.

In the end, what that code represents is message passing to the **SkyLight** private framework, the one that handles the macOS window management, Dock, Spaces and a ton of other stuff. *I’m guessing someone sneaked in a VM debugger or looked through the assembly code to find the right bytes to send.*

Ok, enumeration and focusing is doable, what else do we need? Right, **Accessibility permissions**. Here comes the biggest hurdle.

### How do you escape the macOS sandbox?

You don’t.

On macOS, an app can be run:
- within a sandbox
    - *where it has its own limited view of the file system and limited access to privileged APIs*
- outside the sandbox
    - *where it has access to everything that’s not guarded by SIP (System Integrity Protection)*

**App Store apps can only run inside the sandbox**, and within that, an app can’t ask for Accessibility permissions. The API for that just throws a silent error and does nothing.

But then how does Magnet do it, and a few other apps as well like Peek or PopClip for example?

Turns out, these apps have a special exception from Apple, mostly because they were on the App Store before the sandbox has become mandatory: [objective c - How to use Accessibility with sandboxed app? - Stack Overflow](https://stackoverflow.com/a/32248238)

I can barely get my apps to not be rejected by the App Store reviewers, I’m not going to get an exception just so that rcmd can focus specific windows. So now what?

## Workarounds

I thought, if there was an app running outside the sandbox and listening for rcmd’s `listWindows` and `focusWindow` commands, I might be able to get this working.

I remembered **![hammerspoon icon](/images/icons/hammerspoon_16@2x.webp#x16) [Hammerspoon](https://www.hammerspoon.org)** having [a really complete window management support](https://www.hammerspoon.org/docs/hs.window.html) and it also being scriptable with Lua made it the perfect choice.

HTTP would probably be overkill for this, I knew Hammerspoon had an inter-process communication (IPC) API built-in so I tried to use that.
```objc
static NSString *portName = @"Hammerspoon";
CFMessagePortRef messagePort = CFMessagePortCreateRemote(NULL, (__bridge CFStringRef)portName);
// messagePort is NULL here
```

Well nope, the sandbox doesn’t allow that.

What about the `hs` CLI that Hammerspoon provides, I knew that you could send arbitrary IPC messages using that, right?

Nope again, any process run by a sandboxed app will inherit that sandbox limitations.

Ok fine, HTTP it is! Thankfully Hammerspoon provides an HTTP server and I just need to register a callback and make it listen on a port. Since we’ve already reached this madness, let’s go straight to websockets.

```lua
function rcmdCallbackWS(msg)
    local params = hs.json.decode(msg)
    local response = "{}"

    if params.cmd == "listWindows" then
        response = hs.json.encode(hs.window:allWindows())
    elseif params.cmd == "focusWindow" then
        hs.window.get(params.window):focus()
    end

    return response
end

server = hs.httpserver.new(false, false)
server:setName("rcmd-hammerspoon")
server:setInterface("localhost")
server:setPort(3094)

server:websocket("/ws", rcmdCallbackWS)
server:start()
```

Alright, this seems to work. I can connect to the Hammerspoon websocket, get all windows, and focus windows by their IDs.

Now how do I explain to rcmd users that in order to focus windows, they need to:
- download a zip file from GitHub releases
- install another app with no affiliation to rcmd
- give that app Accessibility permissions
- install a Lua script in the `~/.hammerspoon` directory
- ensure Hammerspoon is kept running all the time

## Automating the workarounds

The App Store guidelines explicitly forbid an app from installing another app or binary to enhance its capabilities.

> 2.4.5 Apps distributed via the Mac App Store have some additional requirements to keep in mind:

> (iv) They may not download or install standalone apps, kexts, additional code, or resources to add functionality or significantly change the app from what we see during the review process.

So I can’t install Hammerspoon automatically *(it would be a bad idea anyway, this is malware behavior)*, but I can try to automate most of the stuff and present it as a 1-button install action.

So I wrote a function to download `Hammerspoon.zip`, unzip it in a temporary folder, move it to `/Applications`, write `init.lua` and `rcmd.lua` inside the `~/.hammerspoon` directory, launch Hammerspoon and wait for the websocket to be available.

The user only has to click an **Install window switcher** button, no big deal.

### Quarantine says “not so fast”

You see, when a sandboxed app downloads a file, the system automatically adds the `com.apple.quarantine` extended attribute to the file.

```awk
> xattr -l ~/Downloads/Hammerspoon.zip
com.apple.quarantine: 0083;62ea4f5c;Safari;3A6D521B-5E0D-4202-80C4-A5EB567DC246
```

This means that macOS GateKeeper will prevent you from launching any downloaded app or running any binary directly from code.

Even if the user tries to launch the downloaded app manually afterwards, it will still fail with the *App can't be opened* error.

{{< img src="hammerspoon-quarantine.jpg" alt="system dialog with hammerspoon not being allowed to launch because of the quarantine attribute" sizes="(min-width: 60em) 810px, 90vw" >}}

No amount of `xattr -cr Hammerspoon.app` will fix this if run from the sandbox.

Great. Scrap the download and install part, split the button into two buttons:
1. **Install Hammerspoon** which only shows text instructions on how to download and install the app manually
2. **Install custom script** which writes the Lua script files to disk

{{< img src="rcmd-install-hs-buttons.png" alt="rcmd menu showing the two install buttons" sizes="(min-width: 60em) 810px, 90vw" >}}

I’ve streamlined this process as much as the sandbox allows me, and after giving the app to some beta testers, every single one of them found it so confusing that they said they would not use it.

And who can blame them, I myself find it too convoluted whenever I test it.

## So is this on the App Store?

Yes, surprisingly. It passed App Review without a single rejection.

I hid the feature behind a **`Try experimental window switching`** red button to deter support emails on the subject, but it’s there for anyone to try and use.

{{< img src="try-experimental-window-switching-button.png" alt="rcmd menu showing the try experimental window switching button " sizes="(min-width: 60em) 810px, 90vw" >}}

After the initial setup, it actually works pretty reliably, and the websocket connection to Hammerspoon is so fast that I don’t ever notice this happens over the network. It feels like a native window switcher to me.

But I wasn't able to create a seamless experience like I did for app switching.

Oh well, at least I solved my own problem and can get back to what I was doing.

One month later.
