NetworkEvents
===============================
[![Travis CI](https://travis-ci.org/pwittchen/NetworkEvents.svg?branch=master)](https://travis-ci.org/pwittchen/NetworkEvents)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-NetworkEvents-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/1392) 
![Maven Central](https://img.shields.io/maven-central/v/com.github.pwittchen/networkevents.svg?style=flat)

Android library listening network connection state and change of the WiFi signal strength with event bus.

It works with any implementation of the Event Bus. In this repository you can find samples with Otto and GreenRobot's bus.

min sdk version = 9

JavaDoc is available at: http://pwittchen.github.io/NetworkEvents

**Note**: There's another, younger project, which allows to accomplish the same tasks with RxJava and Reactive Programming apporach. You can check it out at: https://github.com/pwittchen/ReactiveNetwork.

Contents
--------
- [Overview](#overview)
- [Usage](#usage)
    - [Initialize objects](#initialize-objects)
      - [NetworkEvents customization](#networkevents-customization)
        - [Custom logger](#custom-logger)
        - [Enabling WiFi scan](#enabling-wifi-scan)
        - [Enabling Internet connection check](#enabling-internet-connection-check)
    - [Register and unregister objects](#register-and-unregister-objects)
    - [Subscribe for the events](#subscribe-for-the-events)
- [Examples](#examples)
- [Download](#download)
- [Tests](#tests)
- [Code style](#code-style)
- [License](#license)

Overview
--------

Library is able to detect `ConnectivityStatus` when it changes.

```java
public enum ConnectivityStatus {
  UNKNOWN("unknown"),
  WIFI_CONNECTED("connected to WiFi"),
  WIFI_CONNECTED_HAS_INTERNET("connected to WiFi (Internet available)"),
  WIFI_CONNECTED_HAS_NO_INTERNET("connected to WiFi (Internet not available)"),
  MOBILE_CONNECTED("connected to mobile network"),
  OFFLINE("offline");
  ...
}    
```

In addition, it is able to detect situation when strength of the Wifi signal was changed with `WifiSignalStrengthChanged` event, when we [enable WiFi scanning](#enabling-wifi-scan).

Library is able to detect `MobileNetworkType` when `ConnectivityStatus` changes to `MOBILE_CONNECTED`.

```java
public enum MobileNetworkType {
  UNKNOWN("unknown"),
  LTE("LTE"),
  HSPAP("HSPAP"),
  EDGE("EDGE"),
  GPRS("GPRS");
  ...
}    
```    

Usage
-----

Appropriate permissions are already set in `AndroidManifest.xml` file for the library inside the `<manifest>` tag.
They don't need to be set inside the specific application, which uses library.

### Initialize objects

In your activity add `BusWrapper` field, which wraps your Event Bus. You can use [Otto](http://square.github.io/otto/) as in this sample and then create `NetworkEvents` field.

```java
private BusWrapper busWrapper;
private NetworkEvents networkEvents;
```

Create implementation of `BusWrapper`. You can use any event bus here. E.g. [GreenRobot's Event Bus](https://github.com/greenrobot/EventBus). In this example, we are wrapping Otto Event bus.

```java
private BusWrapper getOttoBusWrapper(final Bus bus) {
  return new BusWrapper() {
    @Override public void register(Object object) {
      bus.register(object);
    }

    @Override public void unregister(Object object) {
      bus.unregister(object);
    }

    @Override public void post(Object event) {
      bus.post(event);
    }
  };
}
```    

Initialize objects in `onCreate(Bundle savedInstanceState)` method.

```java
@Override protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  busWrapper = getOttoBusWrapper(new Bus());
  networkEvents = new NetworkEvents(context, busWrapper);
}
```

**Please note**: Due to memory leak in `WifiManager` reported
in [issue 43945](https://code.google.com/p/android/issues/detail?id=43945) in Android issue tracker
it's recommended to use Application Context instead of Activity Context.

#### NetworkEvents Customization

##### Custom logger

By default library logs messages about changed connectivity or WiFi signal strength to LogCat.
We can create custom logger implementation in the following way:

```java
networkEvents = new NetworkEvents(context, busWrapper, new Logger() {
  @Override public void log(String message) {
    // log your message here
  }
});
```

If we don't want to log anything, we can simply create empty implementation of the `Logger` interface, when `log(message)` method doesn't do anything.

##### enabling WiFi scan

WiFi Access Points scanning is disabled by default. If Wifi Access Points Scan is not enabled, `WifiSignalStrengthChanged` event will never occur. You can enable it as follows:

```java
networkEvents = new NetworkEvents(context, busWrapper)
  .enableWifiScan();
```

##### enabling Internet connection check

Internet connection check is disabled by default. If Internet check is disabled, status `WIFI_CONNECTED_HAS_INTERNET` and `WIFI_CONNECTED_HAS_NO_INTERNET` won't be set.
If internet check is enabled `WIFI_CONNECTED` status will never occur (from version 2.1.0). The only statuses, which may occur after connecting to WiFi after enabling this option are `WIFI_CONNECTED_HAS_INTERNET` and `WIFI_CONNECTED_HAS_NO_INTERNET`.

You can enable internet check as follows:

```java
networkEvents = new NetworkEvents(context, busWrapper)
  .enableInternetCheck();
```

### Register and unregister objects

We have to register and unregister objects in Activity Lifecycle.

In case of different Event Buses, we have to do it differently.

#### Otto Bus

Register `BusWrapper` and `NetworkEvents` in `onResume()` method and unregister them in `onPause()` method.

```java
@Override protected void onResume() {
  super.onResume();
  busWrapper.register(this);
  networkEvents.register();
}

@Override protected void onPause() {
  super.onPause();
  busWrapper.unregister(this);
  networkEvents.unregister();
}
```

#### GreenRobot's Bus

Register `BusWrapper` and `NetworkEvents` in `onStart()` method and unregister them in `onStop()` method.

```java
@Override protected void onStart() {
  super.onStart();
  busWrapper.register(this);
  networkEvents.register();
}

@Override protected void onStop() {
  busWrapper.unregister(this);
  networkEvents.unregister();
  super.onStop();
}
```    

### Subscribe for the events

For Otto Event Bus `@Subscribe` annotations are required, but we don't have to use them in case of using library with GreenRobot's Event Bus.

```java
@Subscribe public void onEvent(ConnectivityChanged event) {
  // get connectivity status from event.getConnectivityStatus()
  // or mobile network type via event.getMobileNetworkType()
  // and do whatever you want
}

@Subscribe public void onEvent(WifiSignalStrengthChanged event) {
  // do whatever you want - e.g. read fresh list of access points
  // via event.getWifiScanResults() method
}
```

Examples
--------

* Look at `MainActivity` in application located in `example` directory to see how this library works with Otto Event Bus.
* If you want to use this library with [Dagger](https://github.com/square/dagger), check `example-dagger` directory.
* Example presenting how to use this library with GreenRobot's Event Bus is presented in `example-greenrobot-bus	` directory

Download
--------

You can depend on the library through Maven:

```xml
<dependency>
  <groupId>com.github.pwittchen</groupId>
  <artifactId>networkevents</artifactId>
  <version>2.1.3</version>
</dependency>
```

or through Gradle:

```groovy
dependencies {
  compile 'com.github.pwittchen:networkevents:2.1.3'
}
```

**Remember to add dependency to the Event Bus, which you are using.**

In case of Otto, add the following dependency through Maven:

```xml
<dependency>
  <groupId>com.squareup</groupId>
  <artifactId>otto</artifactId>
  <version>1.3.8</version>
</dependency>
```

or through Gradle:

```groovy
dependencies {
  compile 'com.squareup:otto:1.3.8'
}
```

You can also use **GreenRobot's Event Bus** or **any Event Bus you want**.

Tests
-----

Tests are available in `network-events-library/src/androidTest/java/` directory and can be executed on emulator or Android device from Android Studio or CLI with the following command:

```
./gradlew connectedCheck
```

Test coverage report can be generated with the following command:

```
./gradlew createDebugCoverageReport
```

In order to generate report, emulator or Android device needs to be connected to the computer.
Report will be generated in the `network-events-library/build/outputs/reports/coverage/debug/` directory.

Code style
----------

Code style used in the project is called `SquareAndroid` from Java Code Styles repository by Square available at: https://github.com/square/java-code-styles.

License
-------

    Copyright 2015 Piotr Wittchen

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
