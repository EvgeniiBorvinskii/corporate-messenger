# 🎨 Настройка смены иконки приложения при смене темы

## ❗ ВАЖНО
Смена иконки приложения на iOS требует специальной настройки в Xcode. Это не может быть сделано автоматически через Flutter.

## 📋 Что нужно сделать:

### Шаг 1: Подготовка иконок

Вам нужны 2 набора иконок (по 3 размера каждый):

**Neon Theme (logo.png):**
- Icon-60@2x.png (120x120)
- Icon-60@3x.png (180x180)  
- Icon-76@2x.png (152x152)

**Classic Theme (logo2.png):**
- Icon-Classic-60@2x.png (120x120)
- Icon-Classic-60@3x.png (180x180)
- Icon-Classic-76@2x.png (152x152)

### Шаг 2: Открыть Xcode

```bash
cd "/Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile/ios"
open Runner.xcworkspace
```

### Шаг 3: Настройка Alternate Icons

1. В Xcode откройте `Runner/Info.plist`
2. Добавьте новый ключ `CFBundleIcons`
3. Внутри добавьте `CFBundleAlternateIcons`
4. Создайте два набора:
   - `ClassicIcon` - для Classic темы
   - `NeonIcon` - для Neon темы

Пример структуры Info.plist:

```xml
<key>CFBundleIcons</key>
<dict>
    <key>CFBundleAlternateIcons</key>
    <dict>
        <key>ClassicIcon</key>
        <dict>
            <key>CFBundleIconFiles</key>
            <array>
                <string>Icon-Classic</string>
            </array>
            <key>UIPrerenderedIcon</key>
            <false/>
        </dict>
        <key>NeonIcon</key>
        <dict>
            <key>CFBundleIconFiles</key>
            <array>
                <string>Icon-Neon</string>
            </array>
            <key>UIPrerenderedIcon</key>
            <false/>
        </dict>
    </dict>
    <key>CFBundlePrimaryIcon</key>
    <dict>
        <key>CFBundleIconFiles</key>
        <array>
            <string>AppIcon</string>
        </array>
    </dict>
</dict>
```

### Шаг 4: Добавить иконки в проект

1. Скопируйте подготовленные иконки в `ios/Runner/Assets.xcassets/`
2. Создайте два новых Icon Set:
   - `Icon-Classic.appiconset`
   - `Icon-Neon.appiconset`
3. Поместите соответствующие изображения в каждый набор

### Шаг 5: Обновить код Flutter

Добавьте в `UserThemeProvider` метод смены иконки:

```dart
Future<void> setTheme(UserVisualTheme theme) async {
  _currentTheme = theme;
  notifyListeners();

  final prefs = await SharedPreferences.getInstance();
  await prefs.setString(_themeKey, theme.id);
  
  // Change app icon on iOS
  if (Platform.isIOS) {
    try {
      final iconName = theme.id == 'classic' ? 'ClassicIcon' : 'NeonIcon';
      await MethodChannel('app_icon').invokeMethod('changeIcon', iconName);
    } catch (e) {
      print('Failed to change app icon: $e');
    }
  }
}
```

### Шаг 6: Создать iOS Channel

В `ios/Runner/AppDelegate.swift` добавьте:

```swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller : FlutterViewController = window?.rootViewController as! FlutterViewController
    let iconChannel = FlutterMethodChannel(name: "app_icon",
                                           binaryMessenger: controller.binaryMessenger)
    
    iconChannel.setMethodCallHandler({
      (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
      if call.method == "changeIcon" {
        guard let iconName = call.arguments as? String else {
          result(FlutterError(code: "INVALID_ARGUMENT",
                            message: "Icon name must be a string",
                            details: nil))
          return
        }
        
        if #available(iOS 10.3, *) {
          UIApplication.shared.setAlternateIconName(iconName == "default" ? nil : iconName) { error in
            if let error = error {
              result(FlutterError(code: "ICON_CHANGE_FAILED",
                                message: error.localizedDescription,
                                details: nil))
            } else {
              result(nil)
            }
          }
        } else {
          result(FlutterError(code: "UNSUPPORTED",
                            message: "iOS version too old",
                            details: nil))
        }
      } else {
        result(FlutterMethodNotImplemented)
      }
    })
    
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

## ⚠️ Ограничения

1. iOS показывает системный диалог при смене иконки (нельзя отключить)
2. Требуется iOS 10.3 или выше
3. Все иконки должны быть включены в сборку приложения
4. Смена происходит только после подтверждения пользователем

## 🎯 Альтернативное решение

**Простое решение**: Использовать одну иконку для приложения и показывать разные темы внутри приложения. Большинство приложений делают именно так.

---

## 📝 Резюме

Смена иконки приложения - сложная функция, требующая:
- Настройки в Xcode
- Создания нескольких наборов иконок
- Platform Channel кода
- Согласия пользователя каждый раз

**Рекомендация**: Оставить одну иконку (текущую Classic), а темы менять только внутри приложения (фон, видео, логотип).
