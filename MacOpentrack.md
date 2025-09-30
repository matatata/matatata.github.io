# MacOpentrack
In the last year I've been investing quite a lot of time in bringing [opentrack](https://github.com/opentrack/opentrack) back to the Mac:

2025
- Introduce *freetrack 2.0 Posix* and the accompanying *freetrackclient* library and SDK for macOS and Linux. It allows native (non-windows) games to support headtracking via Opentrack by loading and using the provided shared library similarly to what happens with the well known protocols on the Windows platform. I invite and encourage game developers to integrate it into their native macOS and Linux games. As an example the xplane-plugin that ships with opentrack now leverages that library as well. The API is designed to be familiar with what they've already done on the Windows platforn.
- I no longer support and include the original WINE output module. Instead I offer a different approach that will use "freetrack 2.0 Posix" and socalled *builtin* wine dlls that will be able to directly read opentrack's headpose. I successfully tested it with stock wine, CrossOver 25 (macOS) and Steam's Proton on Linux. With the original WINE module I encountered a lot of problems. Visit [OpentrackWine](OpentrackWine.md) for more information. 

2024
- Binding key commands was not working properly
- If you had more than one camera connected, a tracker would most of the time not use the one you've selected.
- The X-Plane plugin is now compatible with X-Camera and features a menu that helps you to monitor and control its state.
- I'va made a modification to the output method which was either called X-Plane or Wine depending how you built it from source. I wanted to be able to use choose between relaying the headpose to a windows-game via [Wine](https://www.winehq.org) or to the X-Plane plugin. That's possible now.


## Sourcecode and binaries

I've made the code changes available in [my fork](https://github.com/matatata/opentrack). Some have already been merged to the original codebase.

I now provide codesigned and apple-notarised [pre-built binaries](https://matatata.gumroad.com/l/macopentrack) via gumroad allowing you to support my work.


## Further Reading

I've published a [detailed guide](https://delanclip.com/ir-head-tracking-macos-opentrack-delanclip/) about how proper IR-Head-Tracking is now possible on macOS using products by DelanClip. It performs much much better than any facetracking solution you might have looked into in the past. Get a [5% discount using this link](https://delanclip.com/?a=tomatec).
