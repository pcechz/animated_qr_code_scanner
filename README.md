# Animated QR Code Scanner

A QR code scanner Widget that currently works on **Android only** (feel free to make pull request on iOS implementation) by natively embedding the platform view within Flutter.

This work is based on Julius Canute's [qr_code_scanner](https://pub.dev/packages/qr_code_scanner) Flutter package.

A decorative square is displayed on the middle of the view until a QR code is detected, which will be animated to the location of the QR code to highlight it. Decoded text is passed to callback function when a QR is detected at the beginning and/or the end of highlight animation.

The location of the detected code on the Flutter's display is calculated again with Detector and Perspective Transform from zxing which are ported to Flutter.

A controller can also store the text, and provides basic controls such as turning on flashlight and restarting the scan.

It is also possible to style the targetting box.

## Demo

![Animation Demo](https://raw.githubusercontent.com/kiatuki/animated_qr_code_scanner/master/docs/images/demo.gif)

## Setup

Modify the android-level build.gradle (\android\build.gradle) gradle to 3.3.1
```dart
dependencies {
    classpath 'com.android.tools.build:gradle:3.3.1'
    ...
}
```

Modify the app-level build.gradle (\android\app\build.gradle) minSdkVersion to at least 24

```dart
defaultConfig {
    ...
    minSdkVersion 24
    ...
}
```

## Installing

Use this package as a library
### 1. Depend on it
Add this to your package's pubspec.yaml file:

```dart
dependencies:
  animated_qr_code_scanner:
    git:
      url: https://github.com/kiatuki/animated_qr_code_scanner.git
      ref: v0.1.7
```

### 2. Install it
You can install packages from the command line:

with Flutter:

```dart
$ flutter pub get
```

Alternatively, your editor might support `flutter pub get`. Check the docs for your editor to learn more.

### 3. Import it
Now in your Dart code, you can use:

```dart
import 'package:animated_qr_code_scanner/animated_qr_code_scanner.dart';
```

## Example

When a QR code is detected, print the decoded text on the console and start animating the targetting box to highlight the detected QR code.
Then after the QR code has been highlighted, show a dialog to let user choose whether to re-scan or confirm the code.
A page with the code is displayed if user confirms the code.

```dart
class TestPage extends StatelessWidget {
  final AnimatedQRViewController controller = AnimatedQRViewController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Expanded(
            child: AnimatedQRView(
              squareColor: Colors.green.withOpacity(0.25),
              animationDuration: const Duration(milliseconds: 400),
              onScanBeforeAnimation: (String str) {
                print('Callback at the beginning of animation: $str');
              },
              onScan: (String str) async {
                await showDialog(
                  context: context,
                  builder: (context) => AlertDialog(
                    title: Text("Callback at the end of animation: $str"),
                    actions: [
                      FlatButton(
                        child: Text('OK'),
                        onPressed: () {
                          Navigator.of(context).pop();
                          Navigator.of(context).pop();
                          Navigator.of(context).push(
                            MaterialPageRoute(
                              builder: (context) => Scaffold(
                                body: Align(
                                  alignment: Alignment.center,
                                  child: Text("$str"),
                                ),
                              ),
                            ),
                          );
                        },
                      ),
                      FlatButton(
                        child: Text('Rescan'),
                        onPressed: () {
                          Navigator.of(context).pop();
                          controller.resume();
                        },
                      ),
                    ],
                  ),
                );
              },
              controller: controller,
            ),
          ),
          Container(
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                FlatButton(
                  color: Colors.blue,
                  child: Text('Flash'),
                  onPressed: () {
                    controller.toggleFlash();
                  },
                ),
                const SizedBox(width: 10),
                FlatButton(
                  color: Colors.blue,
                  child: Text('Flip'),
                  onPressed: () {
                    controller.flipCamera();
                  },
                ),
                const SizedBox(width: 10),
                FlatButton(
                  color: Colors.blue,
                  child: Text('Resume'),
                  onPressed: () {
                    controller.resume();
                  },
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

## Flashlight
By default, the flashlight is OFF.
```dart
controller.toggleFlash();
```

## Resume/Pause
Pause camera stream and scanner.
```dart
controller.pause();
```
Resume camera stream and scanner.
```dart
controller.resume();
```

## Styling the Targetting Square
Change the color of the insides and the borders of targetting square, and the animation duration
```dart
AnimatedQRView(
    squareColor: Colors.teal.withOpacity(0.25),
    squareBorderColor: Colors.cyan,
    animationDuration: const Duration(milliseconds: 400),
    ...
),
```

## Todos
QR Detector still detects barcode even with intent changed

Compute the QR corners in native-layer instead of in Flutter-layer

Implementation for iOS

Animation for front camera is incorrect
