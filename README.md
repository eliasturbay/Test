## HockeyTech Ops iOS Client

### Requirements

HockeyTech Ops requires [iOS 9.0](https://developer.apple.com/library/content/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html) and above as SDK. And [XCode 8.0](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/WhatsNewXcode/introduction.html) and above as IDE.

### Getting Started

To clone the project through the command line type:

``` bash
git clone git@github.com:Band-of-Coders/hockeytechopsiosapp.git
```

If you're using GitHub application or SourceTree you should use this URL:

  https://github.com/Band-of-Coders/hockeytechopsiosapp.git

Since this project handles the dependencies with [Cocoapods](https://cocoapods.org/), you'll need to have it installed and run the following to install the third party libraries:

``` bash
pod install
```

#### Run the app

Once you've cloned the project and installed all the dependencies, just open it (the .xcworkspace file) on XCode and run it using the desired iOS Simulator.


#### Run the tests

To run the UI automated tests you can run the following command:

``` bash
xcodebuild -workspace HT\ Ops\ iOS.xcworkspace \
	   	   -scheme "HT Ops iOS" \
           -sdk iphonesimulator \
           -destination 'platform=iOS Simulator,name=iPhone 7,OS=10.0'
           test
```

Remember to run the test before each release, and also to update this document upon every release in case it's needed.


#### App Distribution

First of all you will need to request HockeyTech for access to their Apple Developer Portal to get the right iOS Distribution Certificates, Provisioning Profiles, etc. Then you will be able to create a new archive for Ad Hoc distribution.

The distribution is made through [HockeyApp](https://www.hockeyapp.net/), and you can directly login from (https://rink.hockeyapp.net/users/sign_in). Or you can also use their Desktop app, you can find more information about this at (https://www.hockeyapp.net/features/distribution)

You will also need to ask HockeyTech to create a new developer account in order to be able to login and upload the .ipa file.