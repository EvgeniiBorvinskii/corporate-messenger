# 🔧 Как исправить проблему с Alternate Icons в Xcode

## Проблема
После выбора темы иконки "слетают" потому что iOS не может найти файлы alternate icons.

## Решение - добавить иконки в Xcode вручную

### Шаг 1: Открыть проект в Xcode
```bash
open "/Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile/ios/Runner.xcworkspace"
```

### Шаг 2: Добавить файлы иконок

1. В **Project Navigator** (левая панель) найдите папку **Runner**
2. **Перетащите (drag & drop)** эти 4 файла из Finder в Xcode в папку Runner:
   ```
   /Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile/ios/Runner/AppIcon-Neon@2x.png
   /Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile/ios/Runner/AppIcon-Neon@3x.png
   /Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile/ios/Runner/AppIcon-Classic@2x.png
   /Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile/ios/Runner/AppIcon-Classic@3x.png
   ```

3. В появившемся диалоге:
   - ☑️ **СНИМИТЕ галочку** "Copy items if needed" (файлы уже там)
   - ☑️ **Выберите** "Create groups"
   - ☑️ **Отметьте** "Add to targets: Runner"
   - Нажмите **"Finish"**

### Шаг 3: Проверить что файлы добавлены

1. Выберите **Runner** project (синяя иконка вверху Project Navigator)
2. Выберите **Runner** target
3. Перейдите на вкладку **"Build Phases"**
4. Разверните **"Copy Bundle Resources"**
5. Проверьте что там есть все 4 иконки:
   - AppIcon-Neon@2x.png
   - AppIcon-Neon@3x.png  
   - AppIcon-Classic@2x.png
   - AppIcon-Classic@3x.png

6. Если их нет - нажмите **"+"** внизу списка и добавьте

### Шаг 4: Закрыть Xcode и пересобрать

```bash
cd "/Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile"
flutter clean
flutter build ios --release --no-codesign
```

### Шаг 5: Проверить

1. Установите приложение на устройство
2. Смените тему
3. iOS покажет alert о смене иконки - нажмите OK
4. Перезапустите приложение
5. Иконка должна измениться!

## Если все еще не работает

Проблема может быть в том, что iOS требует ТОЧНЫЕ имена файлов. Проверьте в Info.plist:

```xml
<key>CFBundleAlternateIcons</key>
<dict>
    <key>AppIcon-Neon</key>
    <dict>
        <key>CFBundleIconFiles</key>
        <array>
            <string>AppIcon-Neon</string>
        </array>
    </dict>
    <key>AppIcon-Classic</key>
    <dict>
        <key>CFBundleIconFiles</key>
        <array>
            <string>AppIcon-Classic</string>
        </array>
    </dict>
</dict>
```

Имя в массиве `CFBundleIconFiles` должно совпадать с именем файла БЕЗ @2x/@3x суффикса!

## Альтернатива - отключить смену иконок

Если ничего не помогает, можно просто отключить смену иконок и оставить одну:

В `user_theme_provider.dart` закомментируйте:
```dart
// try {
//   await _changeAppIcon(theme);
// } catch (e) {
//   ...
// }
```

Тема все равно будет видна по splash screen, логотипам и цветам!
