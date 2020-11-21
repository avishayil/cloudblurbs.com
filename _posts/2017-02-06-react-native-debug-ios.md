---
layout: post
title: "Release The Pain from Running and Debugging React Native App on a real iOS Device"
description: "React Native debugging on a real iOS device could be a pain. Here is a simple trick to make your life a little bit easier"
thumb_image: "react-native.png"
tags: [react-native, ios, debug]
canonical_url: "https://medium.com/@avishayil/release-the-pain-from-running-and-debugging-react-native-app-on-a-real-ios-device-7a2e6048609e"
---

{% include image.html path="react-native.png" path-detail="react-native.png" alt="React Native" %}

## Introduction ##

So far, debugging React Native on a real iOS device is a mess: a developer needs to do the next steps:

- Discover the local IP address of the machine running the React Native Packager.
- Update the AppDelegate.m file with the IP address mentioned above.
- Run the app on the real device.

The problem with this scenario, is that sometimes we want to switch environments without messing with the IP addresses. We want to develop rapidly on the iOS Simulator, than switch to a real device, then pull the changes on another machine and keep developing or handing the code to another developer.

At Geektime, we came up with a solution when encountering this mess while developing the Geektime Mobile App (Play Store / App Store). The following solution is applicable for both working with the iOS Simulator and a real iOS device, such as iPhone or iPad.

## Getting Started ##

- Create an empty Key on the main Info.plist file of your React Native Project. Near the Information Property List Key you’ll see a + icon, click on it. Call your Key with the following name: ReactNativeDevServerIP.
- Click on your Project file on the Xcode file tree and go to Build Phases. See the + sign on the upper-left side of the inner box? Click on it and put the following script on the box below the Shell:
  ```bash
  if [ “$CONFIGURATION” == “Debug” ]; then
  echo -n ${TARGET_BUILD_DIR}/${INFOPLIST_PATH} | xargs -0 /usr/libexec/PlistBuddy -c “Set :ReactNativeDevServerIP `ipconfig getifaddr en0`”
  else
  echo -n ${TARGET_BUILD_DIR}/${INFOPLIST_PATH} | xargs -0 /usr/libexec/PlistBuddy -c “Delete :ReactNativeDevServerIP”
  fi
  ```
- On your AppDelegate.m you’ll see the jsCodeLocation variable assigned on the #ifdef DEBUG block, change it to the following:
  ```objc
  jsCodeLocation = [NSURL URLWithString:[NSString stringWithFormat:@”%@%@%@”, @”http://”, [[NSBundle mainBundle] objectForInfoDictionaryKey:@”ReactNativeDevServerIP”], @”:8081/index.ios.bundle?platform=ios&dev=true”]];
  ```
  This string is concentrated with 3 strings:
  - `@"http://"`, which is the protocol of the debugger.
  - `[[NSBundle mainBundle] objectForInfoDictionaryKey:@"ReactNativeDevServerIP"]`, which contains the IP of the current machine, derived from the build step we defined earlier on Build Phases and passed to the Info.plist file during Debug build.
  - `@":8081/index.ios.bundle?platform=ios&dev=true"`, which contains the port of the running React Native Packager and the iOS minified js bundle.

After completing these steps, you should be good to go.

Good luck and enjoy developing with React Native!

Avishay.
