# Paint.NET-on-Wine
If you want to run Paint.NET on Linux or Mac, this is the package to use.

Paint.NET-on-Wine is currently **"experimental."** It's not yet a "beta" or an "alpha," it is definitely "pre-alpha." It is intended for early adopters who are comfortable with sharp edges and willing to report bugs and other issues. I don't know how fast progress will be, as this is a side project and I can't always dedicate a lot of time to it. It may take awhile, please do not ask "is it done yet?" etc.

When or if things move past "experimental," some kind of announcement will be made at the various website links listed below. Whether or not this becomes an "officially" "supported" flavor of Paint.NET is yet to be figured out. The best outcome, of course, is that Wine itself gets all patched up and the need for a special Paint.NET-on-Wine package is entirely eliminated. 

Builds of Paint.NET-on-Wine will expire 12 weeks after their build date. You'll need to get the latest version when that happens. This helps to ensure that old, stale, buggy builds do not survive out in the wild, and reduces time wasted on reporting and triaging things that have already been fixed. The expiration date is displayed near the top of Settings → Diagnostics.

## Download
Grab the latest build on the Releases page: https://github.com/paintdotnet/Paint.NET-on-Wine/releases

## Install
Run `./install.sh`. It will (hopefully) set things up properly and make sure Wine and DXVK are installed, etc.

## Run
Run `./paintdotnet.sh`. It will check some things and then launch `paintdotnet.exe` using Wine.

## Report Bugs
Please report bugs and other issues by creating an Issue on the GitHub page at https://github.com/paintdotnet/Paint.NET-on-Wine. They can't be fixed if you don't tell me about them -- I don't use Linux or Mac myself, so please don't assume that anything you bump into is "known" or "obvious."

Before reporting anything, make sure you're using the latest versions of things like Wine and DXVK. As of September 2026, the absolute bare minimum version of Wine you'll need is 11.15.

## Discussion
The best place for discussing Paint.NET-on-Wine is on the Paint.NET Discord server in the `#pdn-on-wine` channel: https://discord.gg/SMnDg99Mpb

## Technical Details
Paint.NET does not automatically detect Wine. Instead, "Wine mode" is enabled via the `EnableWineMode` property in `PaintDotNet.Configuration.json`. This is set to `true` for the Paint.NET-on-Wine package.

When running Paint.NET in "Wine mode," a few things are different than on native Windows. These avoid holes in Wine that prevent Paint.NET from working:

- The native Direct2D (`d2d1.dll`) is replaced with a from-scratch implementation written in C# that lives in `PaintDotNet.Windows.Direct2D1.Managed.dll`.
- [DWriteCore](https://learn.microsoft.com/en-us/windows/win32/directwrite/dwritecore-overview) (`dwritecore.dll`) is used instead of the system-wide DirectWrite (`dwrite.dll`).
  - DWriteCore is an official version of DirectWrite that is redistributable and intended for cross-platform or down-level scenarios. This removes Wine's implementation of DirectWrite as a dependency and ensures text rendering is consistent and stable.
  - If you'd like to test using the system-wide DirectWrite instead, use the `/useNativeDWrite` command-line parameter.
- Google's [Noto Color Emoji](https://github.com/googlefonts/noto-emoji) font is used for emoji fallback rendering. Segoe UI Emoji usually takes on this role in Windows so that emojis can be used even if a font doesn't support them.
  - If you prefer to use a different font for this purpose, it can be configured in the `PaintDotNet.Configuration.json` file with the `DWriteCoreFallbackFontFileName` and `DWriteCoreFallbackFontFamily` properties. 
  - `DWriteCoreFallbackFontFileName` is relative to the application directory. If the property is missing then font fallback will not be modified -- whatever Wine provides is what you'll get. This can be useful if you want to develop/test upstream font-related fixes for Wine.
  - When `DWriteCoreFallbackFontFamily` is missing or empty, the first font family from the given font file will be used (the default value is an empty string).
- The native Windows Animation Manager (`UIAnimation.dll`) is replaced with a from-scratch implementation written in C# that lives in `PaintDotNet.Windows.Animation.Managed.dll`.
  - You can specify `/useNativeWAM` on the command-line if you'd rather use `UIAnimation.dll` instead. Wine's implementation is full of broken stubs, so this is meant for anyone who wants to work on fixing Wine's `UIAnimation.dll`.
- The use of composition (`Windows.UI.Composition`, related to DirectComposition) is disabled. This prevents the use of a large memory usage optimization related to DXGI swapchain reuse, but doesn't affect correct operation of the app.
- The use of Windows 11's `DisplayInformation` class is disabled. This prevents the detection of Wide Color Gamut mode, which is Automatic Color Management operating in SDR mode. Unless HDR mode is enabled and also works in Wine (which is unknown), Paint.NET will always operate in "sRGB mode." See the in-app Settings → Color Management for more information.
- `GpuEffect` plugins that use [ComputeSharp](https://github.com/Sergio0694/ComputeSharp) are not supported yet. They work against Direct2D's native COM objects, so I just need to add CCW's (COM Callable Wrappers) for the managed Direct2D implementation.
- Crash logs point at the GitHub repository linked above instead of the main crash log e-mail, and when the build expires it will also point to the GitHub repository instead of doing an update check and linking to the main website.
- A wine glass emoji 🍷 is added to the title bar caption area.

Over time, some of these "quirks modes" may no longer be necessary, in which case newer builds will remove them.

"Wine mode" also works on Windows. This can be a useful debugging technique to help determine if an issue is specific to Wine.

## Known Issues
- Performance isn't as good as the native Windows build, but has also been making rapid progress.
- File format issues. Wine doesn't have full implementations for the various file codecs, so you'll often see images that don't load or that don't come with their color profile or other metadata. Over time these can be addressed by either patching up Wine, or reimplementing these internally in Paint.NET, or both.
- Various cosmetic UI issues. The toolbar looks incorrect because of incorrect metrics from Wine, and similar for the Tools window being double width. The non-Microsoft implementations of DirectWrite don't always handle color emojis or have other text rendering blunders, etc.
- Printing won't work because it makes use of WIA (Windows Image Acquisition) and its goofy "Photo Print Wizard" from the Windows XP era. At some point this will be completely replaced with a modern UI, and then printing should also work in Wine.
- Scanning won't work because it also uses WIA and its goofy XP-era scanning UI.
- Automatic Updates won't work because this is a portable build. I will probably add update checking to at least notify you that there's a new build. For now, the expiration mechanism is the only thing enforcing this.

## Links
- Paint.NET-on-Wine GitHub (releases and issue tracking): https://github.com/paintdotnet/Paint.NET-on-Wine
- @bluesillybeard's Paint.NETOnWine GitHub repository for upstream Wine patches: https://github.com/bluesillybeard/Paint.NETOnWine
- Paint.NET website: https://paint.net
- Paint.NET forum: https://forums.paint.net
- Paint.NET blog: https://blog.paint.net
- Paint.NET Discord: https://discord.gg/SMnDg99Mpb
- Rick Brewster on X: http://x.com/rickbrewPDN/
- Rick Brewster on Bluesky: https://bsky.app/profile/rickbrew.bsky.social
