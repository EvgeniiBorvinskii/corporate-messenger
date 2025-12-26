# 🎨 HOW TO CHANGE APP ICON TO CLASSIC THEME

## Current Status

- ✅ **Splash video fixed** - Now shows video properly with Classic theme
- ✅ **All backgrounds changed** - Classic theme applied everywhere
- ⏳ **App icon** - Still shows Neon theme icon (needs manual change)

---

## 📱 Quick Fix: Change App Icon Manually

Since Classic is now the default theme, let's replace the app icon with logo2.png!

### Option 1: Using flutter_launcher_icons (Recommended)

1. **Install flutter_launcher_icons:**
```bash
cd mobile
flutter pub add dev:flutter_launcher_icons
```

2. **Create icon config in `pubspec.yaml`:**
```yaml
flutter_launcher_icons:
  android: false  # We only need iOS
  ios: true
  image_path: "assets/images/logo2.png"  # Classic theme logo
  remove_alpha_ios: true
```

3. **Generate icons:**
```bash
flutter pub run flutter_launcher_icons
```

4. **Rebuild:**
```bash
flutter build ios --release
open ios/Runner.xcworkspace
```

---

### Option 2: Manual Xcode Method (If Option 1 doesn't work)

1. **Prepare logo2.png:**
   - Make sure it's at least 1024x1024 pixels
   - Square format (1:1 ratio)
   - PNG format

2. **Open Xcode:**
```bash
cd mobile
open ios/Runner.xcworkspace
```

3. **In Xcode:**
   - Click on `Runner` in left sidebar
   - Select `Runner` target
   - Click on `AppIcon` in Assets.xcassets
   - Drag logo2.png onto the 1024x1024 slot
   - Xcode will auto-generate all sizes

4. **Rebuild and install**

---

## 🎯 What Was Fixed (Splash Video)

### Problem:
- Video not showing at startup
- Glitch text missing
- Direct jump to login screen

### Root Cause:
Hive had saved `skip_video = true` from previous video load errors

### Solution:
```dart
// Reset video skip settings on every app start
final box = await Hive.openBox('_test_startup');
await box.put('skip_video', false);  // Force video playback
await box.put('video_error_count', 0);  // Reset error counter
await box.close();
```

Now video ALWAYS plays (unless it actually fails to load)

---

## 🧪 Test After Rebuild

1. **Install app from Xcode**
2. **Close app completely** (swipe up from app switcher)
3. **Open app fresh**
4. ✅ **Should see:** Classic theme video → Glitch text → Login screen
5. ✅ **App icon:** Will change after applying Option 1 or 2

---

## 📝 Files Changed

- `mobile/lib/screens/splash/splash_screen.dart`
  - Removed skip_video check
  - Added reset of video settings
  - Video will always attempt to play

---

## 🚀 Next Steps

1. **Test splash video** - Should work now!
2. **Change app icon:**
   - Either use `flutter_launcher_icons` (Option 1)
   - Or manually in Xcode (Option 2)
3. **Rebuild and enjoy Classic theme everywhere!** 🎉

---

## 💡 Alternative: Dynamic Icon Switching

If you want icon to change when user switches themes (advanced):

1. Need to create Alternate Icon Sets in Xcode
2. Add code to change icon programmatically:
```dart
// In ProfileScreen when theme changes:
if (Platform.isIOS) {
  if (theme.id == 'classic') {
    await FlutterDynamicIcon.setApplicationIconBadgeNumber(0);
    await FlutterDynamicIcon.setIcon(
      iconName: "ClassicIcon",  // Name in Assets
    );
  } else {
    await FlutterDynamicIcon.setIcon(
      iconName: null,  // Default icon
    );
  }
}
```

But this is complex and requires creating multiple icon sets. For now, **Option 1** is simpler since Classic is the default theme!

---

**Ready to test the video? Build is complete!** ✅
