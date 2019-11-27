---
layout: post
title: 6 Things to Consider When Developing Push (Remote) Notifications on iOS
---

1. Don't convert the `deviceToken` you get from APNS to **String** using `description` property. Convert the byte sequence manually since `description` doesn't return corresponding hexadecimal representation for applications built against iOS 13 SDK.

2. For simplicity, just implement:
```swift
    func application(
        _ application: UIApplication,
        didReceiveRemoteNotification userInfo: [AnyHashable: Any],
        fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void)
```
to handle received notifications. This method is called whenever you receive a notification when the application is in foreground or background (even when suspended for silent notifications).
* If user opens the application by tapping the notification, method is called again. So you don't need to check for `launchOptions` parameter in `application:didFinishLaunchingWithOptions:`.
* As you might have noticed, this delegate is called multiple times if the user opens the app via notification. First when received, second when application is coming to foreground. Be prepared.
* Don't forget to call `completionHandler` with valid completion parameter when you are finished processing the notification. If you don't call it, the system will stop delivering the silent notifications to your app, since your application is consuming system resources for nothing.

3. Use `content-available` flag to send silent notifications. Even if your application is suspended by the system, it will be woken up in background.

4. You don't need user's permission to receive silent notifications, its enabled by default. Still it can be disabled in **Settings->General->Background App Refresh**.

5. Use `mutable-content` flag to mutate the notification to display rich notifications.
* Don't trust the breakpoints when debugging **Notification Service** extension. 
* Rich notifications doesn't launch the application in background, just the extension gets triggered.

6. Applications signed with Debug provisioning profiles can only use Sandbox APNS environment.
