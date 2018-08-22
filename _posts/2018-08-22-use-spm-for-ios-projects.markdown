---
layout: post
title:  "Swift Package Manager for iOS Projects"
date:   2018-08-22 08:00:00 +0200
---

At [the peak lab][tpl] we develop many different apps for iOS, tvOS, macOS, watchOS as well as server side swift applications.
In the past few years the most common package managers were [CocoaPods][cocoapods] and [Carthage][carthage]. But for some large projects we discovered limitations for example the missing ability to [nest dependencies](https://github.com/Carthage/Carthage/issues/844#issuecomment-147783855) in Carthage.

[Swift Package Manager][spm] is a great tool for managing the dependencies of your project.

> The Swift Package Manager is a tool for managing the distribution of Swift code. It’s integrated with the Swift build system to automate the process of downloading, compiling, and linking dependencies.
> 
> -- <cite>[Apple Inc.][spm]</cite>

The problem with Swift Package Manager is, that it is a Package Manager for **Swift** and not a platform package manager for **iOS**. So there is **no explicit support for depending on UIKit or AppKit**.

Many voices are saying that it is not possible to use Swift Package Manager as a dependencie manager for iOS. Others offer solutions which are not working when it comes to uploading the app to the appstore.

# Real app walkthrough

In this post i'm going to explain how to use Swift Package Manager for iOS projects and which kind of problems i had. I'm explaining this with one of the apps i made: [JP Fan App](https://itunes.apple.com/de/app/jp-fan-app/id1286558522).

The app in my example is really simple and easy to understand. There is a backend service (using [vapor][vapor], also written in swift) running on a small [Digital Ocean][do] droplet. The backend service provides data in form of a REST interface. To access the backend service, my idea was to write a basic HTTP client which wraps the REST interface: **JPFanAppClient**. The good thing: writing this HTTP client using Swift Package Manager enables me to use the Framework on all environments iOS, macOS, tvOS, watchOS, Linux. So its easy for me to use the Framework on all of these platforms as well.

It becomes really interesting when you think about using the same frameworks in the backend and frontend.

# Setup Swift Package Manager

#### My Environment

{% highlight shell %}
> swift --version
Apple Swift version 4.1.2 (swiftlang-902.0.54 clang-902.0.39.2)
Target: x86_64-apple-darwin17.5.0

Xcode: Version 9.4.1 (9F2000)
{% endhighlight %}

We'll start from the project folder where my `.xcodeproj` and `.xcworkspace` files are.
To use Swift Package Manager dependencies, you need to create a small "wrapper" package, generate the Xcode project and do the correct import into your existing iOS Xcode project.

#### Create the Dependencies Package

{% highlight shell %}
# create change directory
mkdir Dependencies
cd Dependencies

# initialize a new package
swift package init --type library
{% endhighlight %}

Add your dependencies to the new generated file `Package.swift`.  
In my case `git@github.com:cpageler93/jpperformance-client.git`.
Consider the part where `JPFanAppClient` must be included in the dependencies array in the `Dependencies` target.
{% highlight swift %}
// swift-tools-version:4.0
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "Dependencies",
    products: [
        .library(name: "Dependencies", targets: ["Dependencies"]),
    ],
    dependencies: [
        .package(url: "git@github.com:cpageler93/jpperformance-client.git", from: "0.7.0"),
    ],
    targets: [
        .target(name: "Dependencies", dependencies: ["JPFanAppClient"]),
        .testTarget(name: "DependenciesTests", dependencies: ["Dependencies"]),
    ]
)
{% endhighlight %}

Fetch your dependencies.

{% highlight shell %}
> swift package update
Fetching git@github.com:cpageler93/jpperformance-client.git
Fetching https://github.com/cpageler93/Quack.git
Fetching https://github.com/Alamofire/Alamofire
Fetching https://github.com/IBM-Swift/SwiftyJSON.git
Fetching https://github.com/antitypical/Result.git
Cloning https://github.com/IBM-Swift/SwiftyJSON.git
Resolving https://github.com/IBM-Swift/SwiftyJSON.git at 17.0.2
Cloning git@github.com:cpageler93/jpperformance-client.git
Resolving git@github.com:cpageler93/jpperformance-client.git at 0.7.0
Cloning https://github.com/antitypical/Result.git
Resolving https://github.com/antitypical/Result.git at 3.2.4
Cloning https://github.com/cpageler93/Quack.git
Resolving https://github.com/cpageler93/Quack.git at 1.8.0
Cloning https://github.com/Alamofire/Alamofire
Resolving https://github.com/Alamofire/Alamofire at 4.7.3

{% endhighlight %}

Your output is maybe much smaller. In my case i'm using the [Quack][quack] HTTP client which wrapps Alamofire/Vapor HTTP and works on all swift platforms as well.

{% highlight shell %}
> swift package show-dependencies
.
└── JPFanAppClient<git@github.com:cpageler93/jpperformance-client.git@0.7.0>
    └── Quack<https://github.com/cpageler93/Quack.git@1.8.0>
        ├── Alamofire<https://github.com/Alamofire/Alamofire@4.7.3>
        ├── SwiftyJSON<https://github.com/IBM-Swift/SwiftyJSON.git@17.0.2>
        └── Result<https://github.com/antitypical/Result.git@3.2.4>

{% endhighlight %}

#### Import into your Xcode project

To import the frameworks into the existing Xcode workspace i'll take advantage of the feature to generate an Xcode Project from my package.

{% highlight shell %}
> swift package generate-xcodeproj
generated: ./Dependencies.xcodeproj

{% endhighlight %}

File > Add Files to "Your Workspace Name"

![Add Files Screenshot]({{site.url}}/assets/spm_ios/1_add_files_to_jpperformance.png){:width="50%"}

Select the new Dependencies.xcodeproj

![Select Dependencies Project]({{site.url}}/assets/spm_ios/2_select_dependencies_project.png){:width="80%"}


Add **all** the frameworks from your **Dependencies** project to **Embedded binaries**.

![Select Dependencies Project]({{site.url}}/assets/spm_ios/3_add_embedded_binaries.png){:width="100%"}

![Select Dependencies Project]({{site.url}}/assets/spm_ios/4_add_all_frameworks.png){:width="100%"}

# First Problems

When importing one of the new dependencies one of the first errors i got was:

<code>
Module file's minimum deployment target is ios11.4 v11.4: /Users/christoph/Library/Developer/Xcode/DerivedData/JPPerformance-cteaudztylounnaqgegjnjqthxbt/Build/Products/Debug-iphonesimulator/SwiftyJSON.framework/Modules/SwiftyJSON.swiftmodule/x86_64.swiftmodule
</code>

![Select Dependencies Project]({{site.url}}/assets/spm_ios/5_first_problem_minimum_deployment_target.png){:width="100%"}


To solve this problem we have to set the deployment target to the matching target of the iOS project. In my case the **JP Fan App iOS Deployment Target is iOS 10**. Since we'll regulary update our dependencies and regenerate the `Dependencies.xcodeproj` it wasn't an option for me to set anything in the project by hand. We'll automate this with a [Rakefile][rake] as described in other "solutions".

**Caution: this is not the final Rakefile** i'll describe the problems with using the other solutions and how i solved them.

{% highlight shell %}
# create Rakefile
touch Rakefile
open Rakefile
{% endhighlight %}

Rakefile
{% highlight ruby %}
#!/usr/bin/env ruby

require 'xcodeproj'

task :dependencies do

  system "swift package update"
  system "swift package generate-xcodeproj"

  project = Xcodeproj::Project.open('Dependencies.xcodeproj')

  project.targets.each do |target|
    module_map_file = "Dependencies.xcodeproj/GeneratedModuleMap/#{target.name}/module.modulemap"

    project.build_configurations.each { |config|
      config.build_settings["SDKROOT"] = "iphoneos"
      config.build_settings["IPHONEOS_DEPLOYMENT_TARGET"] = 10.0
      config.build_settings.delete("SUPPORTED_PLATFORMS")
    }

    target.build_configurations.each do |config|
      config.build_settings['DEFINES_MODULE'] = 'YES'
      if File.exist? module_map_file
        config.build_settings['MODULEMAP_FILE'] = "${SRCROOT}/#{module_map_file}"
      end
    end
  end

  project.save

end
{% endhighlight %}

Calling `rake dependencies` will now update our dependencies, regenerate the `Dependencies.xcodeproj` and change the settings so that they are valid for our iOS Project. **You may need to change** `IPHONEOS_DEPLOYMENT_TARGET` to your desired deployment target. So you'll never have to type `swift package generate-xcodeproj` again, from now on we're using Rake.

# Problems seems to be fixed

Try to rebuild your Xcode project - everything should seem to work, the compiler error disappears. ✅🎉  

The annoying part for me was that the next problems did not occour until compiling for release, when you want to upload the app to [HockeyApp][hockeyapp] or [TestFlight][testflight].

I'm using [fastlane][fastlane] to automate screenshot generation, testing or uploading for beta-testing or deployment to AppStore Connect. So the next error occured when calling `fastlane beta`.

**Compiling**: worked ✅  
**Generating the Archive**: worked ✅  

{% highlight shell %}
...
[14:38:13]: ▸ Running script 'Carthage Copy Frameworks'
[14:38:17]: ▸ Running script '[CP] Embed Pods Frameworks'
[14:38:17]: ▸ Running script '[CP] Copy Pods Resources'
[14:38:17]: ▸ Running script 'Increment Build Number'
[14:38:17]: ▸ Touching JPPerformance.app
[14:38:22]: ▸ Signing /Users/christoph/Library/Developer/Xcode/DerivedData/JPPerformance-cteaudztylounnaqgegjnjqthxbt/Build/Intermediates.noindex/ArchiveIntermediates/JPPerformance/InstallationBuildProductsLocation/Applications/JPPerformance.app
[14:38:24]: ▸ Touching JPPerformance.app.dSYM
[14:38:24]: ▸ Archive Succeeded
{% endhighlight %}

**Exporting the Archive**: Boom 💥  

Somewhere when it comes to the command `xcodebuild -exportArchive -exportOptionsPlist /var/folders/7q/004mxqbj4kn117z_rptqddg80000gn/T/gym_config20180822-62378-1urdrvm.plist -archivePath '/Users/christoph/Library/Developer/Xcode/Archives/2018-08-22/JPPerformance 2018-08-22 14.37.11.xcarchive' -exportPath /var/folders/7q/004mxqbj4kn117z_rptqddg80000gn/T/gym_output20180822-62378-g7reue` i got a bunch of errors and warnings like the following:  

`IDEDistribution: Step failed: <IDEDistributionPackagingStep: 0x7fbf75727ef0>: Error Domain=IDEFoundationErrorDomain Code=1 "ipatool failed with an exception: #<CmdSpec::NonZeroExcitException: /Applications/Xcode.app/Contents/Developer/usr/bin/bitcode-build-tool exited with pid 64585 exit 1`

`warning: using sysroot for 'MacOSX' but targeting 'iPhone'`

`error: libswiftCore.dylib not found in dylib search path`

# Solution

If you want to build your app not just in Xcode for `debug` but also, in `release` to bring the app into the app store you can't avoid the part of exporting the archive.

For me the errors looked like "there is some macOS stuff inside my iOS binary". So i checked the configuration and found quite a few entries:

![Remaining macOS Stuff]({{site.url}}/assets/spm_ios/6_remaining_macos_stuff.png){:width="100%"}


**LD_RUNPATH_SEARCH_PATHS** has a reference to macOS: `$(TOOLCHAIN_DIR)/usr/lib/swift/macosx`
and **MACOSX_DEPLOYMENT_TARGET** is set to the Swift Package Manager's default `macOS 10.10`.

#### Adjusting the Rakefile

By adding two simple lines to the `Rakefile` we can reduce the amount of errors and warning when exporting the archive to: **0** 🎉  

{% highlight ruby %}
config.build_settings["LD_RUNPATH_SEARCH_PATHS"] = "$(inherited) @executable_path/Frameworks"
config.build_settings.delete("MACOSX_DEPLOYMENT_TARGET")
{% endhighlight %}

So the **final Rakefile** looks like this:

{% highlight ruby %}
#!/usr/bin/env ruby

require 'xcodeproj'

task :dependencies do

  system "swift package update"
  system "swift package generate-xcodeproj"

  project = Xcodeproj::Project.open('Dependencies.xcodeproj')

  project.targets.each do |target|
    module_map_file = "Dependencies.xcodeproj/GeneratedModuleMap/#{target.name}/module.modulemap"

    project.build_configurations.each { |config|
      config.build_settings["SDKROOT"] = "iphoneos"
      config.build_settings["IPHONEOS_DEPLOYMENT_TARGET"] = 10.0
      config.build_settings["LD_RUNPATH_SEARCH_PATHS"] = "$(inherited) @executable_path/Frameworks"
      config.build_settings.delete("MACOSX_DEPLOYMENT_TARGET")
      config.build_settings.delete("SUPPORTED_PLATFORMS")
    }

    target.build_configurations.each do |config|
      config.build_settings['DEFINES_MODULE'] = 'YES'
      if File.exist? module_map_file
        config.build_settings['MODULEMAP_FILE'] = "${SRCROOT}/#{module_map_file}"
      end
    end
  end

  project.save

end
{% endhighlight %}

Update your dependencies using `rake dependencies`. From this point on, the export of the archive worked for me and i hope yours will work as well.

For uploads to AppStore Connect you may have to set `CURRENT_PROJECT_VERSION` in your Build Settings.


[spm]: https://swift.org/package-manager
[tpl]: https://www.thepeaklab.com
[cocoapods]: https://cocoapods.org
[carthage]: https://github.com/Carthage/Carthage
[do]: https://www.digitalocean.com
[vapor]: https://www.digitalocean.com
[quack]: https://github.com/cpageler93/quack
[rake]: https://github.com/ruby/rake
[hockeyapp]: https://hockeyapp.net/
[testflight]: https://developer.apple.com/testflight/
[fastlane]: https://fastlane.tools