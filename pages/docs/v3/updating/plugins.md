---
title: Updating Capacitor in your plugin
description: Guide for updating Capacitor in your plugin
---

# Updating Capacitor in your plugin

## Update to 3.0.0 (WIP)

There are several required and recommended changes for plugins that are updating to Capacitor 3.

### Planning for a Core API

It is currently difficult for the core team to make changes to the internals of Capacitor without potentially affecting plugins. Because most classes and methods in Capacitor 2 are public for both iOS and Android, we have observed undesired usage of Capacitor APIs that we considered internal.

During Capacitor 3 development, we will be evaluating this problem and creating an official public API for plugins, which will be [documented here](/docs/core-apis).

### Android

#### Use the new `@CapacitorPlugin` annotation

The `@NativePlugin` annotation is deprecated. We now recommend using the new `@CapacitorPlugin` annotation, which will allow for the [new permissions API](#adopting-the-new-permissions-api).

The `name`, `requestCodes`, and `permissionRequestCode` attributes are the same. The `permissions` will need to be replaced with list of `@Permission` annotations, each containing a list of manifest strings and their corresponding `alias`, which you can omit for now until the new permissions API is implemented in your plugin.

```diff-java
-@NativePlugin(
+@CapacitorPlugin(
     name = "FooBar",
     requestCodes = { 10051, 10052, 10053 },
     permissionRequestCode = 10050,
-    permissions = { Manifest.permission.FOO, Manifest.permission.BAR }
+    permissions = {
+        @Permission(strings = { Manifest.permission.FOO }, alias = "foo"),
+        @Permission(strings = { Manifest.permission.BAR }, alias = "bar")
+    })
 )
 public class FooBar extends Plugin {

 }
```

#### Evaluate Android request codes

It may be a good time to evaluate the request codes your plugin is using. Since almost all app interaction occurs in a single activity, request codes must be unique. The following ranges are a guideline to mitigate this potential issue:

- <100 should be reserved for Android
- 100-8999 should be reserved for apps
- 9000-9999 should be reserved for Capacitor core and official plugins
- 10000+ should be reserved for community/third-party plugins

### iOS

TODO

### Changes to `PluginCall` & `CAPPluginCall`

#### Use `resolve()` and `reject()`

We believe `resolve()` and `reject()` better reflect the Promise-like flow intended for plugin methods. They should be preferred over `success()` and `error()` (now deprecated), even in callback-style plugin methods.

#### `resolve()` without arguments now resolves with `undefined`

Previously, calling `resolve()` with no arguments resulted in an empty object being sent to the JavaScript layer. Because this is unlike the behavior of JavaScript's `Promise.resolve()`, as of Capacitor 3, `undefined` is sent instead.

### Web

TODO

### Evaluate errors and error codes

TODO

### Adopting the new Permissions API

Prior to 3.0, it was expected that permissions were automatically requested by a plugin before feature use. For example, a Geolocation plugin would automatically request permission when the user location was requested for the first time and then continue appropriately if the permission was granted or denied.

One goal of Capacitor 3 is to give app developers the ability to request or check permissions at any time and control how and when the user prompts are presented. This provides more flexibility in the user experience by allowing the app to respond to the user's choice in a variety of ways.

It is perfectly fine to continue automatically requesting permissions, but you are encouraged to adopt the new permissions pattern as well to give app developers control over permissions.

TODO: link to guide