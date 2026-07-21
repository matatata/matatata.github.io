# MacOpentrack
I've been investing quite a lot of time in bringing [opentrack](https://github.com/opentrack/opentrack) back to the Mac. Support for macOS already existed for a long time, but with time it got more and more orphaned and there were no pre-built binaries available. So I've been trying to revive the macOS port. All my changes are available in [my fork](https://github.com/matatata/opentrack).

**DOWNLOAD I provide codesigned and apple-notarised pre-built binaries via** [gumroad](https://matatata.gumroad.com/l/macopentrack) allowing you to $$support$$ my work. You can of course build opentrack from sources, but to compile all interesting modules is not trivial and very time-consuming.

## Releases:

### Version 2026.1.1-matatata.4
- Includes upstream changes from the upstream 2026.1.0 release.
  - Most notably it now uses the QT6 framework wich fixes the QT5 Bug where Slider-Controls behave have visual glitches when editing the Filter.

- I created a new output module named *opentrackclient 1.0* which is accompanied by a client library that allows game developers to easily integrate opentrack into their games.  Opentrack Wine Bridge and the X-Plane plugin are using it. It could be used for other macOS Software/Games as well. I is an abstraction, so that developers need not to deal with opentrack's internal workings. Currently this most likely will not work for sandboxed applications (like the ones distributed via Apple’s AppStore). As a future goal this approach allows to change the interprocess communication method from posix shared memory to something that will be more compatible with Apple"s restrictions. Of course the UDP networking could be used, which existed for years.

- I had to remove the old *Wine/X-Plane Output* protocol due to changes in Wine since version 10.12. That’s why I’ve created an [alternative solution called Opentrack Wine Bridge](https://matatata.gumroad.com/l/opentrackwinebridge). It has the benefit of even working with CrossOver. **X-Plane-Users can simply switch to opentrackclient 1.0** as the X-Plane plugin now uses the aforemention opentrackclient library. I you rely on the old module keep using 2024.1.1-matatata.1.

- The OSC (Open Sound Control) module is now included.

### Version 2024.1.1-matatata.1
- Based on [opentrack-2024.1.1](https://github.com/opentrack/opentrack/releases/tag/opentrack-2024.1.1)
- Binding key commands was not working properly
- If you had more than one camera connected, a tracker would most of the time not use the one you've selected.
- The X-Plane plugin is now compatible with X-Camera and features a menu that helps you to monitor and control its state.
- I'va made a modification to the output method which was either called X-Plane or Wine depending how you built it from source. I wanted to be able to use choose between relaying the headpose to a windows-game via [Wine](https://www.winehq.org) or to the X-Plane plugin. That's possible now.

## General Info
I hav published a [detailed guide](https://delanclip.com/ir-head-tracking-macos-opentrack-delanclip/) about how proper IR-Head-Tracking is now possible on macOS using products by DelanClip. It performs much much better than any facetracking solution you might have looked into in the past. Get a [5% discount using this link](https://delanclip.com/?a=tomatec).
