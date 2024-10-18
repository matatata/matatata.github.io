# MacOpentrack
In the last year I've been investing quite a lot of time in bringing [opentrack](https://guthub.com/opentrack/opentrack) back to the Mac:

- Binding key commands was not working properly
- If you had more than one camera connected, a tracker would most of the time not use the one you've selected.
- The X-Plane plugin is now compatible with X-Camera and features a menu that helps you to monitor and control its state.
- I'va made a modification to the output method which was either called X-Plane or Wine depending how you built it from source. I wanted to be able to use choose between relaying the headpose to a windows-game via [Wine](https://www.winehq.org) or to the X-Plane plugin. That's possible now.

I've made the code changes available in [my fork](https://github.com/matatata/opentrack). Some have already been merged to the original codebase.

I now provide codesigned and apple-notarised [pre-built binaries](https://matatata.gumroad.com/l/macopentrack) via gumroad allowing you to support my work.

I've also published a [detailed guide](https://delanclip.com/ir-head-tracking-macos-opentrack-delanclip/) about how proper IR-Head-Tracking is now possible on macOS using products by DelanClip. It performs much much better than any facetracking solution you might have looked into in the past. Get a [5% discount using this link](https://delanclip.com/?a=tomatec).
