This is a fork of [a great plugin by katzer](https://github.com/katzer/cordova-plugin-background-mode/). It aims to keep up-to-date with Android changes while also providing more features.


## Installation
The plugin can be installed via [Cordova-CLI][CLI] and will be publicly available on [NPM][npm] eventually.

Execute from the projects root folder:

    $ cordova plugin add https://github.com/FWiner/cordova-plugin-android-background-mode


## Usage
The plugin creates the object `cordova.plugins.backgroundMode` and is accessible after the *deviceready* event has been fired.

```js
document.addEventListener('deviceready', function () {
    // cordova.plugins.backgroundMode is now available
}, false);
```

### Enable the background mode
The plugin is not enabled by default. Once it has been enabled the mode becomes active if the app moves to background.

```js
cordova.plugins.backgroundMode.enable();
// or
cordova.plugins.backgroundMode.setEnabled(true);
```

To disable the background mode:
```js
cordova.plugins.backgroundMode.disable();
// or
cordova.plugins.backgroundMode.setEnabled(false);
```

### Check if running in background
Once the plugin has been enabled and the app has entered the background, the background mode becomes active.

```js
cordova.plugins.backgroundMode.isActive(); // => boolean
```

A non-active mode means that the app is in foreground.

### Listen for events
The plugin fires an event each time its status has been changed. These events are `enable`, `disable`, `activate`, `deactivate` and `failure`.

```js
cordova.plugins.backgroundMode.on('EVENT', function);
```

To remove an event listeners:
```js
cordova.plugins.backgroundMode.un('EVENT', function);
```


## Android specifics

### Transit between application states
Android allows to programmatically move from foreground to background or vice versa. 

Note: starting with Android 10, you must request the "Draw on Top" permission from the user or the call to `moveToForeground` will silently fail. You can request it with `cordova.plugins.backgroundMode.requestForegroundPermission();`. This permission isn't necessary for `moveToBackground`

```js
cordova.plugins.backgroundMode.moveToBackground();
// or
cordova.plugins.backgroundMode.moveToForeground();
```

### Back button
Override the back button on Android to go to background instead of closing the app.

```js
cordova.plugins.backgroundMode.overrideBackButton();
```

### Recent task list
Exclude the app from the recent task list works on Android 5.0+.

```js
cordova.plugins.backgroundMode.excludeFromTaskList();
```

### Detect screen status
The method works async instead of _isActive()_ or _isEnabled()_.

```js
cordova.plugins.backgroundMode.isScreenOff(function(bool) {
    ...
});
```

### Unlock and wake-up
A wake-up turns on the screen while unlocking moves the app to foreground even the device is locked.

```js
// Turn screen on
cordova.plugins.backgroundMode.wakeUp();
// Turn screen on and show app even locked
cordova.plugins.backgroundMode.unlock();
```

### Request to disable battery optimizations
Starting in Android 8, apps can be put to sleep to conserve battery. When this happens (usually after 5 minutes or so), the background task is killed. This will cause things like MQTT connections to break.

This method will show a permission prompt for the user (only if the app hasn't been granted permission) to ignore the optimization.

```js
cordova.plugins.backgroundMode.disableBatteryOptimizations();
```

You can also open the battery optimization settings menu directly, and get the user to set it manually. This may be a better option for devices which may ignore the prompt above.

```js
cordova.plugins.backgroundMode.openBatteryOptimizationsSettings();
```

To check if battery optimizations are disabled for the app:

```js
cordova.plugins.backgroundMode.isIgnoringBatteryOptimizations(function(isIgnoring) {
    ...
})
```

Additionally, you may find that your JS code begins to run less frequently, or not at all while in the background. This can be due to the webview slowing down its execution due to being in the background. The `disableWebViewOptimizations` function can prevent that, but it's important that it is run _after_ the app goes to the background.

```js
cordova.plugins.backgroundMode.on('activate', function() {
    cordova.plugins.backgroundMode.disableWebViewOptimizations();
});
```

### Notification
To indicate that the app is executing tasks in background and being paused would disrupt the user, the plug-in has to create a notification while in background - like a download progress bar.

#### Override defaults
The title, text and icon for that notification can be customized as below. Also, by default the app will come to foreground when tapping on the notification. That can be changed by setting resume to false. On Android 5.0+, the color option will set the background color of the notification circle. Also on Android 5.0+, setting hidden to false will make the notification visible on lockscreen.

```js
cordova.plugins.backgroundMode.setDefaults({
    title: String,
    text: String,
    subText: String, // see https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setSubText(java.lang.CharSequence)
    icon: 'icon', // this will look for icon.png in platforms/android/res/drawable|mipmap
    color: String, // hex format like 'F14F4D'
    resume: Boolean,
    hidden: Boolean,
    bigText: Boolean,
    channelName: String, // Shown when the user views the app's notification settings
    channelDescription: String, // Shown when the user views the channel's settings
    allowClose: Boolean, // add a "Close" action to the notification
    closeIcon: 'power', // An icon shown for the close action
    closeTitle: 'Close', // The text for the close action
    showWhen: Boolean, //(Default: true) Show the time since the notification was created
    visibility: String, // Android only: one of 'private' (default), 'public' or 'secret' (see https://developer.android.com/reference/android/app/Notification.Builder.html#setVisibility(int))
})
```

To modify the currently displayed notification
```js
cordova.plugins.backgroundMode.configure({ ... });
```

__Note:__ All properties are optional - only override the things you need to.

#### isOpenNotification
To check the notification status for the app:

```js
cordova.plugins.backgroundMode.isOpenNotification(function(status){
    //true or false
});
```

#### openNotificationSettings
Open notification settings for the app:

```js
cordova.plugins.backgroundMode.openNotificationSettings();
```

#### Run in background without notification
In silent mode the plugin will not display a notification - which is not the default. Be aware that Android recommends adding a notification otherwise the OS may pause the app.

```js
cordova.plugins.backgroundMode.setDefaults({ silent: true });
```


## Quirks

Various APIs like playing media or tracking GPS position in background might not work while in background even the background mode is active. To fix such issues the plugin provides a method to disable most optimizations done by Android/CrossWalk.

```js
cordova.plugins.backgroundMode.on('activate', function() {
   cordova.plugins.backgroundMode.disableWebViewOptimizations();
});
```

__Note:__ Calling the method led to increased resource and power consumption.


## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request


## License

This software is released under the Apache 2.0 License.

Made with :yum: from Leipzig.

Modify with FWinner.


