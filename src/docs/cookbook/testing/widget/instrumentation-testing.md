---
title: Live Testing and Firebase Test Lab
prev:
  title: Tap, drag, and enter text
  path: /docs/cookbook/testing/widget/tap-drag
---

In addition to running on your host machine, widget tests can be
run on a live device (physical or emulated). Running on a live
device allows your tests to behave more like integration tests:
they can exercise native code and invoke plugins.

Live widget testing is possible using Flutter driver to launch the
app, or by running a test manually with `flutter run -t`.
In this recipe, learn how to write a test that uses the
`instrumentation_a` adapter and run it locally and on the Firebase
Test Lab service.

This recipe uses the following steps:

  1. Add a dependency on the `instrumentation_adapter` package.
  2. Add a test file that uses the instrumentation adapter test binding.
  3. Add a boilerplate Java instrumentation test file.
  4. Run a WidgetTester test on a locally connected Android device or emulator.
  5. Run a WidgetTester test on Firebase Test Lab.

### 1. Enable the instrumentation adapter test binding.

Add a dependency on the instrumentation_adapter package in the dev_dependencies section of
pubspec.yaml. For plugins, do this in the pubspec.yaml of the example app.

### 2. Enable the instrumentation adapter test binding.

Import 'package:instrumentation_adapter/instrumentation.adapter.dart'.

Invoke `InstrumentationAdapterFlutterBinding.ensureInitialized() at the
start of a widget test file's main().

Ensure that all your tests use `testWidgets` rather than `test`.

### 3. Add a boilerplate Java instrumentation test file.

Because the instrumentation adapter is currently packaged as a plugin and isn't
included in the main Flutter SDK, you'll have to follow a few extra steps to enable
instrumentation builds in your app. First, create a file at
example/android/app/src/androidTest/java/com/example/myapp/MainActivityTest.java
with the following contents, replacing com, example, and myapp with your values:

```
package com.example.myapp;

import androidx.test.rule.ActivityTestRule;
import dev.flutter.plugins.instrumentationadapter.FlutterRunner;
import org.junit.Rule;
import org.junit.runner.RunWith;

@RunWith(FlutterRunner.class)
public class MainActivityTest extends FlutterTest {
  @Rule
  ActivityTestRule<MainActivity> rule = new ActivityTestRule<>(MainActivity.class);
}
```

### 4. Run the test locally on an Android emulator or device.

You can run your tests locally to get results quickly. However, you'll need a
locally connected Android device or an emulator installed. If you don't have one,
you can skip this step.

Run the following command line:

```
./gradlew \
    -Ptarget=<path_to_test>.dart \
    connectedAndroidTest
```

### 5. Run the test in the cloud using Firebase Test Lab.

You're now ready to run your test on Firebase Test Lab. Ensure that you've built
the necessary artifacts for uploading:

./gradlew assembleAndroidTest
./gradlew assembleDebug -Ptarget=<path_to_test>.dart
```

Use gcloud to upload your instrumentation test to Firebase Test Lab

```
gcloud auth activate-service-account --key-file=${HOME}/gcloud-service-key.json
gcloud --quiet config set project flutter-infra
gcloud firebase test android run --type instrumentation \
  --app build/app/outputs/apk/debug/app-debug.apk \
  --test build/app/outputs/apk/androidTest/debug/app-debug-androidTest.apk\
  --timeout 2m \
  --results-bucket=gs://<YOUR_RESULTS_BUCKET> \
  --results-dir=<YOUR_RESULTS_DIR>
```

Once your Firebase Test Lab test is completed, your results will be visible
on the command line.

```
Instrumentation testing complete.

More details are available at (link).
+---------+------------------------+---------------------+
| OUTCOME |    TEST_AXIS_VALUE     |     TEST_DETAILS    |
+---------+------------------------+---------------------+
| Passed  | walleye-26-en-portrait | 1 test cases passed |
+---------+------------------------+---------------------+

All Firebase Test Lab tests successful!
```

You can log in to the Firebase console to download
videos, browse test results, and more.

#### Additional examples

The [Flutter plugins](https://github.com/flutter/plugins) and
[Flutterfire](https://github.com/FirebaseExtended/flutterfire)
repositories contain more examples of Flutter apps that use
the instrumentation adapter to run their tests on Firebase Test
Lab.
