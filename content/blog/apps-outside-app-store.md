---
title: "Why aren't the most useful Mac apps on the App Store?"
subtitle: "A case study into developing an app for the Mac App Store, and all the limitations I ran into while doing that"
excerpt: |-
    You might notice that a very small number of your installed apps come from the App Store.

    While developing [rcmd](https://lowtechguys.com/rcmd), a simple app that I really wanted to publish on the App Store, I ran into a lot more limitations than I was prepared for.

    Here's my story of how I overcame those limitations and then tried to understand why other useful apps chose the self-publishing route.
description: While developing a simple app that I really wanted to publish on the App Store, I ran into a lot more limitations than I was prepared for. This is a story of how I overcame those limitations and then tried to understand why other useful apps chose the self-publishing route.
date: 2021-12-03T19:28:39+02:00
author: Alin Panaitiu
draft: false
image: outside-app-store.png
images:
-  /images/outside-app-store.png
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

Let’s set the stage first. So, it’s Tuesday night and I’m `Command` `Tab`-ing my way through 10 different apps, some with 3-4 windows, while trying to patch bugs in **![Lunar Icon](/images/icons/lunar_16@2x.webp#x16) [Lunar](https://lunar.fyi)** faster than the users can submit the reports. I’m definitely failing. 

I feel my brain pulsing and my ring finger going numb on the `Tab` key. I stop switching apps and just stare at the Xcode window, containing what I knew was Swift code but looked like gibberish right now. 

*“Feels like burnout”* I think. Wasn’t that what I ran away from when I quit my job to make apps for a living?

---

I heard a joke recently:

{{< rawhtml >}}
<details style="margin-top: 0">
    <summary><b>Show joke</b></summary>
    <img src="https://files.alinpanaitiu.com/9to5-joke.webp" alt="Didn't want a 9 to 5 job, now I work 24/7"/>
</details>
{{< /rawhtml >}}

It’s probably only funny for a small group of workaholics, but the reality of those words struck me in the middle of the hysterical laughter I was trying to stop. 

Why am I still developing this app? 

Why am I adding all the features the users are asking for, then deal with the flood of frustrated emails saying *“what an overcomplicated stupid app, I just want to make my screen brighter!!”*, then try to hide advanced features to make it simpler, then get assaulted with the confused *“I can’t change volume anymore fix this ASAP!!!”* because UI changes can very easily introduce bugs by simply forgetting to bind a slider to a value, then get back to scotch taping broken parts slower than the users can report them?

Those features should have probably been their own independent app.

I start to feel my fingers again, press `Command` `Tab` once more, and while looking at the list of app icons I realise something. 

Maybe pressing `Tab` 4-5 times while visually assessing if the selected app icon is the one I want to focus, isn’t the best solution for this kind of workflow. 

So what does my brain do when I feel burnt out? Gives me ideas for even more apps…

## rcmd

That’s how the idea of **![rcmd Icon](/images/icons/rcmd_16@2x.webp#x16) [rcmd](https://lowtechguys.com/rcmd/)** began. We have two Command keys on a Mac keyboard, and the right hand side one is almost never used. What if I use it exclusively for switching apps?

{{< img src="dynamic-rcmd.png" alt="rcmd app screenshot" sizes="(min-width: 60em) 810px, 90vw" >}}


When I used Windows for reverse engineering malware, I liked switching apps using `Win` + `Number` where the number meant the position of the app icon in the taskbar. I didn’t like counting apps however. 

Using the app name felt the most natural. I remembered using [Contexts](https://contexts.co/) for a while, which provides a Spotlight like search bar for fuzzy searching your running apps. But that needed a bit more key presses than I wanted *(that is **1**)* and more attention than I wanted to give *(which is none)*. 

My idea sounded a bit simpler: `Right Command` + `the first letter of the app name`

So simple that people were offended by it...

![hacker news comment screenshot where someone is offended by the price](https://files.alinpanaitiu.com/hn-comment-rcmd-price-offended.webp)

I pitched this idea to **Ovidiu Rusu**, a very good friend of mine, who surprisingly seemed to have the same need as me. We created the first prototype in about a week *(icons and graphics take so much time…)*  and started using it in our day to day work to see if it made sense. 

In less than a day, rcmd became so ingrained in our app switching that we got incredibly annoyed when we had to quit the app for recompiling and debugging. We just kept pressing `Right Command` `X` and staring at the screen like complete idiots, not understanding why Xcode wasn't being focused.

---
 
What most people overlook when they have a simple idea is that 80% of the effort goes into handling edge cases that are not visible in the original idea. 

Just for this simple app we had to solve the following problems:
- What do I do when there are multiple running apps with the same first letter?
    - How do I decide which one gets priority so that I meet user expectations?
    - How do I get to the other apps with less priority in as few key presses as possible?
- How do I persist key assignments if necessary?
- What if the user assigned the key to an app that’s no longer running?
    - Should I launch the app when the key is pressed?
    - Where is the app located?
- There are dozens of running apps while only about 10% of them are actual apps launched by the user. How do I filter those out? The others could be:
    - Menu bar apps *(e.g. Alfred, Lunar etc)*
    - OS services *(e.g. Spotlight, Siri etc.)*
    - Daemons *(e.g. CoreAudio etc)*
- **How do I only listen to the Right Command key?**

This last question is what led me to write this article. It turned out we needed to do quite a few hacks if we wanted to publish this app in the App Store. 

## The Sandbox
Every app that is submitted to the App Store must be compiled to run within a sandbox. This means that the app will run in a container which will have the same structure as your home directory, but with mostly empty folders. 
The sandbox also limits what APIs you can use, and which system components you can communicate with. 

The defacto way of reacting to `Right Command` + `some other key` is to monitor all key events *(yes, just like a keylogger)*, and discard events that don’t contain the Right Command modifier flag. 
```swift
public extension NSEvent.ModifierFlags {
    static let rightCommand = NSEvent.ModifierFlags(rawValue: UInt(NX_DEVICERCMDKEYMASK))
}

NSEvent.addGlobalMonitorForEvents(matching: .keyDown) { event in
    guard event.modifierFlags.contains(.rightCommand) else { return }

    // do your thing
}
```
Easy peasy, right? Well no, because that’s [not allowed on the App Store](https://stackoverflow.com/questions/32116095/how-to-use-accessibility-with-sandboxed-app).

To use that API you need to first request **Accessibility Permissions** from the user. Those permissions are **prohibited inside the Sandbox**, because with those permissions, an app would be able to do all kinds of nasty stuff:
- Log all your key presses *(including passwords if you aren’t using a Secure Input field)*
- Extract text from all the running apps
- Click on buttons inside other apps
- Write text in fields or send key combinations to the system
- Render elements like buttons and tooltips over the interface of other apps

Those are perfectly reasonable things in the context of assistive software, because you need the computer to do stuff for you when you aren’t able to use a keyboard or a mouse/trackpad. 

And you need the computer to read out text from other apps, or show choice buttons which you can trigger with your voice. 

---

[Technical content ahead. Click to skip this section if you're not interested in macOS internals.](#the-app-store-review)

But for rcmd’s use case, we’re restricted to APIs that don’t require these permissions. APIs so old that 64-bit wasn’t even a thing when they launched and they require passing C function pointers instead of our beloved powerful Swift closures. 

That’s the [Carbon API](https://en.wikipedia.org/wiki/Carbon_(API)) and it goes a little something like this:
#### Registering a Command+R hotkey
```swift
// Install the key event handler
var pressedEventType = EventTypeSpec()
pressedEventType.eventClass = OSType(kEventClassKeyboard)
pressedEventType.eventKind = OSType(kEventHotKeyPressed)

InstallEventHandler(GetEventDispatcherTarget(), { _, inEvent, _ -> OSStatus in
    return handlePressedKeyboardEvent(inEvent!)
}, 1, &pressedEventType, nil, nil)


// Register the hotkey
let hotKeyId = EventHotKeyID(signature: UTGetOSTypeFromString("some-unique-identifier" as CFString), id: 0)
var carbonHotKey: EventHotKeyRef?

RegisterEventHotKey(UInt32(kVK_ANSI_R),
                    UInt32(cmdKey),
                    hotKeyId,
                    GetEventDispatcherTarget(),
                    0,
                    &carbonHotKey)

// Handle the event
func handlePressedKeyboardEvent(_ event: EventRef) -> OSStatus {
    assert(Int(GetEventClass(event)) == kEventClassKeyboard, "Unknown event class")

    var hotKeyId = EventHotKeyID()
    let error = GetEventParameter(event,
                                  EventParamName(kEventParamDirectObject),
                                  EventParamName(typeEventHotKeyID),
                                  nil,
                                  MemoryLayout<EventHotKeyID>.size,
                                  nil,
                                  &hotKeyId)

    guard error == noErr else { return error }
    assert(hotKeyId.signature == UTGetOSTypeFromString("some-unique-identifier" as CFString), "Invalid hot key id")

    switch GetEventKind(event) {
    case EventParamName(kEventHotKeyPressed):
        // do your thing.. eventually
    default:
        assert(false, "Unknown event kind")
    }
    return noErr
}
```

***Not so pretty as the `NSEvent` method, but does the job. Kind of.***

You see, that beautiful code macaroni above only lets us listen to `Any Command` + `R`, not specifically the `Right Command`. There's no way to pass something like `rightCmdKey` into `RegisterEventHotKey`.

A workaround I found for this was: 
* Listen for `flagsChanged` 
* Set a global boolean to `true` when the there's a `rightCommand` modifier
* Discard the event if the boolean is `false`

```swift
var rcmd = false

NSEvent.addGlobalMonitorForEvents(matching: .flagsChanged) { event in
    rcmd = event.modifierFlags.contains(.rightCommand)
}

func handleHotkey(key: String) {
    guard rcmd else { return }

    focusApp(with: key)
}
```


*Doing this reminded me of the days I worked with Rust, and how wonderfully impossible a task like this would be. I don't think I'm touching it again, I like my global atomic booleans.*

Now the weirdest limitation hits me. There's no way to discard a hotkey event and forward it back to the system so it can use it for the next handler.

Say I register `Command` `C` and I only want to do something when `Right Command` is held. If I do nothing when `Left Command` is held, then you can't copy text anymore using `Command` `C`.

I tried returning the inappropriately named `OSStatus(eventNotHandledErr)` but the event still doesn't return to the handler chain.

**At this point we seriously considered dropping the App Store idea and just going the self publishing route.**

But just thinking what we would have to do for that triggered something akin to PTSD.

Here's a list with what I can remember off the top of my head from Lunar:
 - Implement the Paddle SDK for licensing
 - Add Sentry for error reporting
 - Lose the useful App Store analytics
 - Lose ratings and reviews
 - Add Sparkle for auto-updating
     - Generate signing keys
     - Create DMG with the app
     - Sign every single update
     - Try not to lose the signing key
     - Generate the appcast XML
     - Host that appcast somewhere
 - Host the app bundle for download somewhere
 
Finding yet another workaround seemed much easier.

Thankfully it really was easy. It turns out that `RegisterEventHotKey` is plenty fast. So fast that we were able to register the hotkeys only when `Right Command` was being held, and unregister them when the key was released.

```swift
import Atomics

var _rcmd = ManagedAtomic<Bool>(false)
var rcmd: Bool {
    get { _rcmd.load(ordering: .relaxed) }
    set { _rcmd.store(newValue, ordering: .sequentiallyConsistent) }
}

NSEvent.addGlobalMonitorForEvents(matching: .flagsChanged) { event in
    rcmd = event.modifierFlags.contains(.rightCommand)

    if rcmd {
        registerHotkeys()
    } else {
        unregisterHotkeys()
    }
}
```

---

## The App Store review

Now rcmd was ready for publishing on the App Store. 

There was one little thing that bothered me though. I usually keep 4-5 separate projects open in Sublime Text, each with its own window. Because of the sandbox, there's no way to get a list of windows for an app and, say, focus a specific one, or cycle between them.

But I found a little gem while I was customising my fork of [yabai](https://github.com/koekeishiya/yabai), a way to trigger Exposé for a single app:

```swift
CoreDockSendNotification("com.apple.expose.front.awake" as CFString)
```

{{< img src="app-window-expose.png" alt="app window expose screenshot" sizes="(min-width: 60em) 810px, 90vw" >}}

We decided to show Exposé if for example you press `rcmd` `s` while Sublime Text is already focused. It was good enough for us.

Not for the App Store reviewer though. 

{{< img src="coredock-review.png" alt="app store review rejecting the expose feature" sizes="(min-width: 60em) 810px, 90vw" >}}

I knew private and undocumented APIs are not seen well on the App Store. But I had no idea they will guarantee a rejection.

## Free Trials

**I like breaking the norm with my creations.** Some of them will be flukes, some will be criticised into oblivion, but a small number of them might turn out to be something a lot of people wanted but didn't know they needed.

rcmd is one of those things: a bit quirky, unique in its approach, and incredibly useful for a specific group of people.

That is also its weak point though. It's hard to communicate this usefulness without being able to try the app first. But as it turns out, the App Store doesn't provide any support for creating a *free XX-day trial* before buying an app.

**Free trials for non-subscription apps have been allowed since mid-2018 on the App Store**, and are supposed to be implemented using in-app purchases. Unfortunately, this approach has a lot of inconveniences which are very well detailed in this article: [Ersatz Free Trials | Bitsplitting.org](https://bitsplitting.org/2018/06/06/ersatz-free-trials/)

These are the biggest shortcomings for my case:
- The trial doesn't start immediately
    - The users have to be nagged with a custom *Activate Trial* UI to start the trial manually
    - Starting the trial means **buying a $0.00 in-app purchase**
    - If the users skip the trial UI out of frustration, they'll be left with a dumbed down version of the app, possibly missing the functionality they downloaded it for
- You have to create your own UI for starting a trial, buying the full app after the trial ends, showing the trial status etc.
- You have to provide some kind of limited functionality if the trial ends
    - In the context of simple and small apps like rcmd, this doesn't really make sense

I tried a few dozen apps on the App Store and I couldn't find a single one offering a free trial for a non-subscription purchase using the above method.

Having to pay upfront is steering away a lot of possible users, but with all that bad UX, we decided to not implement any free trial and just sell the app for a one-time fair price.

---

*3 years ago, I would have probably chosen to make the app open source and give it away for free, just like I did with Lunar.*

*I would have thought:*
> *I'm making a ton of money at this company, what I would get by selling a small app would be peanuts anyway.*

*Only recently I realised that this approach kept me dependent on having a job where I click-clack useless programs 8 hours a day, only to get 1-2 hours after work for my projects, and sacrifice my health and sanity in the process.*

*In my whole 7-year career as a professional API Glue Technician and experienced Wheel Reinventer, I never did anything remotely as useful as even the simplest app I can code and publish in 2 months right now. At those companies, most of my work was scraped anyway when the redesign period of the year came.*

*So I'd rather have those peanuts please.*

---

## Everyone's choosing to be left out

{{< rawhtml >}}
    <style type="text/css">
        #everyones-choosing-to-be-left-out ~ h3 {
            margin-bottom: 0;
            padding-bottom: 0;
            padding-top: 3rem;
            display: flex;
            justify-content: start;
            align-items: center;
        }
        #everyones-choosing-to-be-left-out ~ h3 > img {
            border-radius: 0;
            filter: none;
            margin: 0;
            display: inline;
            object-fit: cover;
            margin-right: 8px;
            height: 1.5rem;
            filter: drop-shadow(0px 2px 3px rgba(10,0,20,0.3));
        }
    </style>
{{< /rawhtml >}}

Now, with so many limitations, I think we can take a fair guess at why most indie developers choose to distribute their app outside the App Store.

Here are some of the apps I find most useful, and what I think is the main reason for them not being in the App Store:

### ![Alfred Icon](/images/icons/alfred.webp#x20) [Alfred](https://www.alfredapp.com/)
The app's main functionality (searching the filesystem) needs **Full Disk Access** permissions which are not allowed inside the sandbox.

It also uses **Accessibility Permissions** for auto-expanding snippets and other custom workflows.

### ![BetterTouchTool Icon](/images/icons/btt.webp#x20) [BetterTouchTool](https://folivora.ai/)
Capturing and responding to all kinds of keyboard and trackpad events needs **Accessibility Permissions**.

The app also encapsulates the older BetterSnapTool utility for snapping windows to a grid. Resizing windows requires the same permissions.

### ![Karabiner-Elements Icon](/images/icons/karabiner.webp#x20) [Karabiner-Elements](https://karabiner-elements.pqrs.org/)
Reacting to and changing keyboard input in realtime needs a special keyboard driver which is only allowed by Apple on a case by case basis. You have to [request DriverKit entitlements from Apple](https://developer.apple.com/documentation/driverkit/requesting_entitlements_for_driverkit_development), and they have to deem you worthy of those entitlements.

Needless to say, they won't give hardware driver entitlements for a software app mimicking a keyboard.

### ![Sublime Text Icon](/images/icons/sublime.webp#x20) [Sublime Text](https://www.sublimetext.com/)
**Full Disk Access** is probably the biggest requirement here. 

Of course, there are other code editors on the App Store like [BBEdit](https://apps.apple.com/us/app/bbedit/id404009241?mt=12) but they have this initial phase where you have to manually give them access to your `/` (root) directory.

{{< img src="bbedit-sandbox-access-dialog.png" alt="bbedit sandbox access dialog" sizes="(min-width: 60em) 810px, 90vw" >}}
{{< img src="bbedit-allow-access.png" alt="bbedit allow access" sizes="(min-width: 60em) 810px, 90vw" >}}

Compared to Sublime Text's ***launch and edit instantly*** first time experience, I feel this is a bit annoying. I'm pretty sure this confuses a lot of first time users, and they will probably blame the developer, not knowing that this is the only way to access files from the Sandbox.

### ![Swish Icon](/images/icons/swish.webp#x20) [Swish](https://highlyopinionated.co/swish/)
Resizing windows, listening for global trackpad gestures, detecting titlebars, moving windows to other spaces/screens. All of these need **Accessibility Permissions**.

There's even an FAQ for that on their page:
 
> ### Why is Swish not on the App Store?
> Apple only allows sandboxed apps on the App Store. Swish needs to perform low-level system operations which prevent it from being sandboxed. [Read more here](https://stackoverflow.com/q/32116095).

### ![Sip Icon](/images/icons/sip.webp#x20) [Sip](https://sipapp.io/)

As outlined in their 2017 article, [Moving from Mac App Store](https://sipapp.medium.com/moving-from-mac-app-store-b41c9e2f53e8), the sandbox limitation is the primary reason

### ![CleanShot X Icon](/images/icons/cleanshot.webp#x20) [CleanShot X](https://cleanshot.com/)
Their Screen Recording feature has three very useful functions:
- **Audio recording** (this needs installing an **audio loopback** device)
- **Pressed keys overlay** (needs **Input Monitoring** permissions)
- **Select single window** (needs **Accessibility Permissions** for getting the window position and size)

### ![Sketch Icon](/images/icons/sketch.webp#x20) [Sketch](https://www.sketch.com/)
Honestly, I'm not sure about this one. The App Store is full of image editors and graphic content creation tools. 

I thing the unique pricing model is something they would have a hard time implementing on the App Store.

*The unique pricing model of Sketch*
{{< img src="sketch-pricing.png" alt="sketch pricing model" sizes="(min-width: 60em) 810px, 90vw" >}}

### ![Parallels Desktop Icon](/images/icons/parallels.webp#x20) [Parallels Desktop](https://www.parallels.com/eu/products/desktop/)
They actually have an App Store edition, but it's severely limited.

Sharing things between the host and the VM is probably the largest functionality affected by the sandbox.

They provide a table with everything that's missing in their App Store version of the app: [KB Parallels: What is the difference between Parallels Desktop App Store Edition and Standard Edition?](https://kb.parallels.com/123796)

### ![Lunar Icon](/images/icons/lunar.webp#x20) [Lunar](https://lunar.fyi/)
Low-level communication with monitors is only possible by using a lot of private and reverse engineered APIs (`IOKit`, `DisplayServices`, `IOAVService` etc.)

**Accessibility Permissions** are also needed for listening and reacting to brightness and volume key events.

Because of the sandbox, the [lite App Store version of Lunar](https://apps.apple.com/us/app/lunar-lite-screen-brightness/id1594174299) only supports software dimming and can only react to F1/F2 keys.

### ![Soulver Icon](/images/icons/soulver.webp#x20) [Soulver](https://soulver.app/)
I think the **free trial limitation** is the only thing keeping such a self-contained app outside the App Store.

Soulver is incredibly complex and useful in its functionality, but I don't think too many people would splurge $35 on a notepad-calculator app without trying it first. It deserves every single dollar of that price, that I can say for sure.

