# pod install 警告梳理

## [!] Unable to read the license file `LICENSE` for the spec `HelperModule (0.0.38)`

* 原因：
    无法读LICENSE文件，


## [!] CocoaPods did not set the base configuration of your project because your project already has a custom config set. In order for CocoaPods integration to work at all, please either set the base configurations of the target `Runner` to `Target Support Files/Pods-Runner/Pods-Runner.betarelease.xcconfig` or include the `Target Support Files/Pods-Runner/Pods-Runner.betarelease.xcconfig` in your build configuration (`Flutter/Release.xcconfig`).

## [!] The `Runner [Release]` target overrides the `ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES` build setting defined in `Pods/Target Support Files/Pods-Runner/Pods-Runner.release.xcconfig'. This can lead to problems with the CocoaPods installation
    - Use the `$(inherited)` flag, or
    - Remove the build settings from the target.

## [!] Found multiple specifications for `Mantle (1.5.6)`:- /Users/bkdevops/.cocoapods/repos/master/Specs/5/d/c/Mantle/1.5.6/Mantle.podspec.json- /Users/bkdevops/.cocoapods/repos/trunk/Specs/5/d/c/Mantle/1.5.6/Mantle.podspec.json

* 原因：
    源冲突，~/.cocoapods/repos 文件夹下有两个源, 一个叫trunk，一个叫master，这俩源中都有相同的pod库，pod install命令不知道要引用哪个，一般不影响使用，但会提示警告，影响观感。

* 解决:
    trunk or master 删掉一个，哪个都行。


## [!] The `Runner [Profile]` target overrides the `ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES` build setting defined in `Pods/Target Support Files/Pods-Runner/Pods-Runner.profile.xcconfig'. This can lead to problems with the CocoaPods installation