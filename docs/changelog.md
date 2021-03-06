# Changelog
## 0.7.5
+ DRAM training was added to fusee-secondary, courtesy @hexkyz.
  + This greatly improves the speed of memory accesses during boot, resulting in a boot time that is ~200-400% faster.
+ creport has had its code region detection improved.
  + Instead of only checking one of the crashing thread's PC/LR for code region presence, creport now checks both + every address in the stacktrace. This is also now done for every thread.
    + This matches the improvement Nintendo added to official creport in 6.1.0.
  + The code region detection heuristic was further improved by checking whether an address points to .rodata or .rwdata, instead of just .text.
  + This means that a crash appears in a loaded NRO (or otherwise discontiguous) code region, creport will be able to detect all active code regions, and not just that one. 
## 0.7.4
+ [libstratosphere](https://github.com/Atmosphere-NX/libstratosphere) has been completely refactored/rewritten, and split into its own, separate submodule.
  + While this is mostly "under the hood" for end-users, the refactor is faster (improving both boot-time and runtime performance), more accurate (many of the internal IPC structures are now bug-for-bug compatible with Nintendo's implementations), and significantly more stable (it fixes a large number of bugs present in the old library).
  + The refactored API is significantly cleaner and easier to write system module code for, which should improve/speed up development of stratosphere.
  + Developers looking to write their own custom system modules for the Switch can now easily include libstratosphere as a submodule in their projects.
+ Loader was extended to add a new generic way to redirect content (ExternalContentSources), courtesy @misson20000:
  + A new command was added to ldr:shel, taking in a tid to redirect and returning a session handle.
  + When the requested TID is loading, Loader will query the handle as though it were an IFileSystem.
    + This allows clients to generically define their own filesystems, and override content with them in loader.
+ fs.mitm has gotten several optimizations that should improve its performance and stability:
  + RomFS redirection now only occurs when there is content to redirect, even if the title is being mitm'd elsewhere.
  + A cache is now maintained of the active data storage, if any, for all opened title IDs. This means if two processes both try to open the same archive, fs.mitm won't duplicate any of its work.
  + RomFS metadata is now cached to the SD card on build instead of being persisted in memory -- this greatly reduces memory footprint and allows fs.mitm to redirect more titles simultaneously than before.
+ A number of bugs were fixed, including:
  + A resource leak was fixed in process creation. This fixes crashes that occur when a large number (>32) games have been launched since the last reboot.
  + fs.mitm no longer errors when receiving a zero-sized buffer. This fixes crashes in some games, including The Messenger.
  + Multi-threaded server semantics should no longer cause deadlocks in certain circumstances. This fixes crashes in some games, including NES Classics.
  + PM now only gives full FS permissions to the active KIPs. This fixes a potential crash where new processes might be unable to be registered with FS.
+ The `make dist` target now includes the branch in the generated zip name.
+ General system stability improvements to enhance the user's experience.
## 0.7.3
+ Loader and fs.mitm now try to reload loader.ini before reading it. This allows for changing the override button combination/HBL title id at runtime.
+ Added a MitM between set:sys and qlaunch, used to override the system version string displayed in system settings.
  + The displayed system version will now display `<Actual version> (AMS <x>.<y>.<z>)`.
+ General system stability improvements to enhance the user's experience.
## 0.7.2
+ Fixed a bug in fs.mitm's LayeredFS read implementation that caused some games to crash when trying to read files.
+ Fixed a bug affecting 1.0.0 that caused games to crash with fatal error 2001-0106 on boot.
+ Improved filenames output by the make dist target.
+ General system stability improvements to enhance the user's experience.

## 0.7.1
+ Fixed a bug preventing consoles on 4.0.0-4.1.0 from going to sleep and waking back up.
+ Fixed a bug preventing consoles on < 4.0.0 from booting without specific KIPs on the SD card.
+ An API was added to Atmosphère's Service Manager for deferring acquisition of all handles for specific services until after early initialization is completed.
+ General system stability improvements to enhance the user's experience.

## 0.7.0
+ First official release of Atmosphère.
+ Supports the following featureset:
  + Fusée, a custom bootloader.
    + Supports loading/customizing of arbitrary KIPs from the SD card.
    + Supports loading a custom kernel from the SD card ("/atmosphere/kernel.bin").
    + Supports compile-time defined kernel patches on a per-firmware basis.
    + All patches at paths like /atmosphere/kip_patches/<user-defined patch name>/<SHA256 of KIP>.ips will be applied to the relevant KIPs, allowing for easy distribution of patches supporting multiple versions.
      + Both the IPS and IPS32 formats are supported.
    + All patches at paths like /atmosphere/kernel_patches/<user-defined patch name>/<SHA256 of Kernel>.ips will be applied to the kernel, allowing for easy distribution of patches supporting multiple versions.
      + Both the IPS and IPS32 formats are supported.
    + Configurable by editing BCT.ini on the SD card.
    + Atmosphère should also be launchable by the alternative hekate bootloader, for those who prefer it.
  + Exosphère, a fully-featured custom secure monitor.
    + Exosphere is a re-implementation of Nintendo's TrustZone firmware, fully replicating all of its features.
    + In addition, it has been extended to provide information on current Atmosphere API version, for homebrew wishing to make use of it.
  + Stratosphère, a set of custom system modules. This includes:
    + A loader system module.
      + Reimplementation of Nintendo's loader, fully replicating all original functionality.
      + Configurable by editing /atmosphere/loader.ini
      + First class support for the Homebrew Loader.
        + An exefs NSP (default "/atmosphere/hbl.nsp") will be used in place of the victim title's exefs.
        + By default, HBL will replace the album applet, but any application should also be supported.
      + Extended to support arbitrary redirection of executable content to the SD card.
        + Files will be preferentially loaded from /atmosphere/titles/<titleid>/exefs/, if present.
        + Files present in the original exefs a user wants to mark as not present may be "stubbed" by creating a .stub file on the SD.
        + If present, a PFS0 at /atmosphere/titles/<titleid>/exefs.nsp will fully replace the original exefs.
        + Redirection is optionally toggleable by holding down certain buttons (by default, holding R disables redirection).
      + Full support for patching NSO content is implemented.
        + All patches at paths like /atmosphere/exefs_patches/<user-defined patch name>/<Hex Build-ID for NSO to patch>.ips will be applied, allowing for easy distribution of patches supporting multiple firmware versions and/or titles.
        + Both the IPS and IPS32 formats are supported.
      + Extended to support launching content from loose executable files on the SD card, without requiring any official installation.
        + This is done by specifying FsStorageId_None on launch.
    + A service manager system module.
      + Reimplementation of Nintendo's service manager, fully replicating all original functionality.
      + Compile-time support for reintroduction of "smhax", allowing clients to optionally skip service access verification by skipping initialization.
      + Extended to allow homebrew to acquire more handles to privileged services than Nintendo natively allows.
      + Extended to add a new API for installing Man-In-The-Middle listeners for arbitrary services.
        + API can additionally be used to safely detect whether a service has been registered in a non-blocking way with no side-effects.
        + Full API documentation to come.
    + A process manager system module.
      + Reimplementation of Nintendo's process manager, fully replicating all original functionality.
      + Extended to allow homebrew to acquire handles to arbitrary processes, and thus read/modify system memory without blocking execution.
      + Extended to allow homebrew to retrieve information about system resource limits.
      + Extended by embedding a full, extended implementation of Nintendo's boot2 system module.
        + Title launch order has been optimized in order to grant access to the SD card faster.
        + The error-collection system module is intentionally not launched, preventing many system telemetry error reports from being generated at all.
        + Users may place their own custom sysmodules on the SD card and flag them for automatic boot2 launch by creating a /atmosphere/titles/<title ID>/boot2.flag file on their SD card.
    + A custom fs.mitm system module.
      + Uses Atmosphère's MitM API in order to provide an easy means for users to modify game content.
      + Intercepts all FS commands sent by games, with special handling for commands used to mount RomFS/DLC content to enable easy creation and distribution of game/DLC mods.
        + fs.mitm will parse the base RomFS image for a game, a RomFS image located at /atmosphere/titles/<title ID>/romfs.bin, and all loose files in /atmosphere/titles/<title ID>/romfs/, and merge them together into a single RomFS image.
        + When merging, loose files are preferred to content in the SD card romfs.bin image, and files from the SD card image are preferred to those in the base image.
      + Can additionally be used to intercept commands sent by arbitrary system titles (excepting those launched before SD card is active), by creating a /atmosphere/titles/<title ID>/fsmitm.flag file on the SD card.
      + Can be forcibly disabled for any title, by creating a /atmosphere/titles/<title ID>/fsmitm_disable.flag file on the SD card.
      + Redirection is optionally toggleable by holding down certain buttons (by default, holding R disables redirection).
    + A custom crash report system module.
      + Serves as a drop-in replacement for Nintendo's own creport system module.
      + Generates detailed, human-readable reports on system crashes, saving to /atmosphere/crash_reports/<timestamp>_<title ID>.log.
      + Because reports are not sent to the erpt sysmodule, this disables all crash report related telemetry.
  + General system stability improvements to enhance the user's experience.
