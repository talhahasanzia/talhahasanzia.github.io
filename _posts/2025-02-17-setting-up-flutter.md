---
title: Setting up Flutter with Android Studio
date: 2025-02-17 19:00:00 +0000
description: Simple setup for flutter
categories: [Flutter]
tags: ['flutter', 'dart']
---

Flutter is a powerful UI toolkit for building natively compiled applications for mobile, web, and desktop from a single codebase. This guide walks you through setting up Flutter with Android Studio to kickstart your app development journey.

## Prerequisites
Before installing Flutter, ensure you have the following:
- A computer running macOS, Windows, or Linux
- Android Studio installed
- A stable internet connection
ß
## Step 1: Installing Dart
Dart is the programming language used by Flußtter. Flutter comes bundled with Dart, but you may need to install it separately for additional tooling.

### Windows:
1. Download the Dart SDK from the [official Dart website](https://dart.dev/get-dart).
2. Extract the downloaded zip file.
3. Add Dart to your system `PATH` by modifying the environment variables.

### macOS:
Run the following command in the terminal:
```sh
brew install dart
```

## Step 2: Installing Flutter
Now, let's install Flutter on your system.


### macOS:
```sh
brew install --cask flutter
```


After installation, verify the installation by running:
```sh
flutter doctor
```
This command checks dependencies and provides recommendations for missing components.

## Step 3: Setting Up Flutter in Android Studio
1. Open Android Studio.
2. Navigate to **Plugins** (File → Settings → Plugins on Windows/Linux, Preferences → Plugins on macOS).
3. Search for **Flutter** and click **Install**.
4. Install the Dart plugin if prompted.
5. Restart Android Studio.

### Creating a New Flutter Project
1. Click **Start a new Flutter project**.
2. Select **Flutter Application**.
3. Set the Flutter SDK path (the directory where Flutter was extracted/installed).
4. Provide a project name and set other configurations.
5. Click **Finish** to generate the project.

### Running Your Flutter App
To run your first Flutter app:
1. Connect a physical device or start an emulator.
2. Open a terminal inside Android Studio.
3. Run:
   ```sh
   flutter run
   ```
4. Your Flutter app should now launch on the selected device/emulator.

## Conclusion
Congratulations! You’ve successfully set up Flutter with Android Studio. You can now start building and testing Flutter applications seamlessly. Happy coding!

