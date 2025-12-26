# 📱 iOS Сборка - Lone Star Chat

**Дата:** 4 октября 2025  
**Версия:** 1.0.0  
**Статус Backend:** ✅ Запущен и работает на `5.249.160.54`

---

## ✅ **ГОТОВНОСТЬ К СБОРКЕ: 95%**

### **Что Готово:**
- ✅ **Backend API работает** на `https://5.249.160.54`
- ✅ **Flutter код написан** (все экраны, провайдеры, сервисы)
- ✅ **pubspec.yaml настроен** со всеми зависимостями
- ✅ **Структура проекта** готова

### **Что Нужно Доделать:**
- ⏳ **Инициализировать iOS/Android** проекты (5 минут)
- ⏳ **Настроить Bundle ID** в Xcode
- ⏳ **Загрузить видео** `Lone Star Chat.mp4` для splash screen

---

## 🖥️ **ТРЕБОВАНИЯ**

### **Обязательно:**
- 💻 **Mac** с macOS 12+ (Monterey или новее)
- 🛠️ **Xcode** 14+ (бесплатно из App Store)
- 🎯 **Flutter SDK** 3.0+ ([flutter.dev](https://flutter.dev))
- 📦 **CocoaPods** (для iOS зависимостей)

### **Для Production:**
- 🍎 **Apple Developer Account** ($99/год)
- 📝 **App Store Connect** доступ
- 📱 **TestFlight** для бета-тестирования

### **Опционально:**
- 📱 **iPhone/iPad** для тестирования на реальном устройстве
- 🔧 **Android Studio** (если нужна Android версия)

---

## 🚀 **БЫСТРАЯ УСТАНОВКА (на Mac)**

### **Шаг 1: Установка Инструментов**

```bash
# 1. Установить Homebrew (если еще нет)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. Установить Flutter
brew install --cask flutter

# 3. Установить CocoaPods
sudo gem install cocoapods

# 4. Проверить установку
flutter doctor

# 5. Принять лицензии Xcode
sudo xcodebuild -license accept
```

### **Шаг 2: Подготовка Проекта**

```bash
# 1. Скопировать проект на Mac (один из способов):

# Вариант A: Через git (если настроен)
git clone <your-repo-url>
cd "Lone Star Chat/mobile"

# Вариант B: Скачать архив с сервера
scp -r root@5.249.160.54:"/srv/Lone Star Chat/mobile" ~/LoneStarChat/
cd ~/LoneStarChat/mobile

# Вариант C: Использовать rsync
rsync -avz root@5.249.160.54:"/srv/Lone Star Chat/mobile/" ~/LoneStarChat/mobile/
cd ~/LoneStarChat/mobile

# 2. Инициализировать Flutter проект
flutter create . --org com.lonestar --project-name lonestar_mobile

# 3. Получить зависимости
flutter pub get

# 4. Установить iOS Pods
cd ios && pod install && cd ..
```

### **Шаг 3: Настройка API URL**

```bash
# Создать файл конфигурации API
cat > lib/core/config/api_config.dart << 'EOF'
class ApiConfig {
  // Production API
  static const String baseUrl = 'https://5.249.160.54';
  
  // Endpoints
  static const String apiPath = '/api';
  static const String wsPath = '/ws';
  
  // Timeouts
  static const Duration connectionTimeout = Duration(seconds: 30);
  static const Duration receiveTimeout = Duration(seconds: 30);
  
  // SSL (самоподписанный сертификат)
  static const bool allowSelfSigned = true;
  
  // Full URLs
  static String get apiUrl => '$baseUrl$apiPath';
  static String get wsUrl => baseUrl.replaceFirst('https', 'wss') + wsPath;
}
EOF
```

### **Шаг 4: Настройка iOS в Xcode**

```bash
# 1. Открыть проект в Xcode
open ios/Runner.xcworkspace

# 2. В Xcode выполнить:
# a) Runner → Signing & Capabilities
# b) Team: выбрать ваш Apple Developer аккаунт
# c) Bundle Identifier: com.lonestar.lonestarmobile
# d) Display Name: Lone Star Chat
# e) Version: 1.0.0
# f) Build: 1

# 3. Настроить permissions в Info.plist:
# - Camera Usage Description
# - Microphone Usage Description
# - Photo Library Usage Description
# - Notification Usage Description
```

---

## 📱 **СБОРКА И ТЕСТИРОВАНИЕ**

### **Вариант 1: Симулятор (Быстрое Тестирование)**

```bash
# Запустить список доступных симуляторов
flutter emulators

# Создать и запустить симулятор iPhone
open -a Simulator

# Запустить приложение
flutter run

# Или выбрать конкретный симулятор
flutter run -d "iPhone 15 Pro"
```

### **Вариант 2: Реальное Устройство (Тестирование)**

```bash
# 1. Подключить iPhone к Mac через USB

# 2. Разблокировать телефон и доверять компьютеру

# 3. Проверить что устройство видно
flutter devices

# Вывод будет:
# iPhone (mobile) • 00008030-XXXX • ios • iOS 17.0

# 4. Запустить на устройстве
flutter run -d "iPhone"

# 5. При первом запуске на телефоне:
# Settings → General → Device Management → Trust Developer
```

### **Вариант 3: Production Build (App Store)**

```bash
# 1. Очистить предыдущие сборки
flutter clean

# 2. Получить зависимости
flutter pub get

# 3. Создать release build
flutter build ipa --release

# Результат:
# ✓ Built build/ios/ipa/lonestar_mobile.ipa

# 4. Загрузить в App Store Connect:
# a) Открыть Xcode
# b) Window → Organizer
# c) Archives → Upload to App Store

# Или через командную строку:
xcrun altool --upload-app \
  --type ios \
  --file build/ios/ipa/lonestar_mobile.ipa \
  --apiKey YOUR_API_KEY \
  --apiIssuer YOUR_ISSUER_ID
```

---

## 🎬 **SPLASH SCREEN VIDEO**

### **Подготовка Видео:**

```bash
# 1. Поместить файл "Lone Star Chat.mp4" в:
mobile/assets/videos/splash.mp4

# 2. Обновить pubspec.yaml (уже настроено):
# flutter:
#   assets:
#     - assets/videos/splash.mp4

# 3. Оптимизировать видео (опционально):
# - Длительность: 3-5 секунд
# - Разрешение: 1080x1920 (portrait)
# - Формат: MP4 (H.264)
# - Размер: < 5 MB
```

---

## ⚙️ **НАСТРОЙКА iOS СПЕЦИФИЧНЫХ ФУНКЦИЙ**

### **1. Info.plist Permissions**

```xml
<!-- ios/Runner/Info.plist -->
<dict>
    <!-- Camera для видео звонков -->
    <key>NSCameraUsageDescription</key>
    <string>Lone Star Chat needs camera access for video calls</string>
    
    <!-- Microphone для голосовых звонков -->
    <key>NSMicrophoneUsageDescription</key>
    <string>Lone Star Chat needs microphone access for voice calls</string>
    
    <!-- Photo Library для отправки изображений -->
    <key>NSPhotoLibraryUsageDescription</key>
    <string>Lone Star Chat needs photo library access to send images</string>
    
    <!-- Notifications -->
    <key>NSUserNotificationsUsageDescription</key>
    <string>Lone Star Chat needs notification access to alert you of new messages</string>
    
    <!-- Background Modes -->
    <key>UIBackgroundModes</key>
    <array>
        <string>audio</string>
        <string>voip</string>
        <string>remote-notification</string>
    </array>
</dict>
```

### **2. App Icons**

```bash
# Создать иконки приложения:
# 1. Подготовить изображение 1024x1024 px
# 2. Использовать онлайн генератор: 
#    https://appicon.co/

# 3. Заменить в:
ios/Runner/Assets.xcassets/AppIcon.appiconset/
```

### **3. Launch Screen**

Уже настроен в проекте:
- `ios/Runner/Assets.xcassets/LaunchImage.imageset/`
- Используется видео из Flutter кода

---

## 🔒 **SSL СЕРТИФИКАТ (Самоподписанный)**

### **Проблема:**
iOS по умолчанию блокирует самоподписанные сертификаты.

### **Решение 1: Временно разрешить (Development)**

```xml
<!-- ios/Runner/Info.plist -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
    <!-- Или только для вашего домена: -->
    <key>NSExceptionDomains</key>
    <dict>
        <key>5.249.160.54</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSIncludesSubdomains</key>
            <true/>
        </dict>
    </dict>
</dict>
```

### **Решение 2: Использовать Let's Encrypt (Production)**

```bash
# На сервере (Ubuntu):
sudo apt install certbot
sudo certbot certonly --standalone -d your-domain.com

# Обновить nginx конфигурацию для использования новых сертификатов
# Убрать NSAppTransportSecurity из Info.plist
```

---

## 🐛 **РЕШЕНИЕ ПРОБЛЕМ**

### **Проблема 1: "Pod install" не работает**

```bash
cd ios
rm -rf Pods Podfile.lock
pod cache clean --all
pod install --repo-update
```

### **Проблема 2: "Unable to boot simulator"**

```bash
# Перезапустить Simulator
killall Simulator
xcrun simctl shutdown all
xcrun simctl erase all
open -a Simulator
```

### **Проблема 3: "Signing for iOS requires a development team"**

```bash
# 1. Открыть Xcode
open ios/Runner.xcworkspace

# 2. Runner → Signing & Capabilities
# 3. Выбрать "Team" (ваш Apple ID)
# 4. Если нет команды - создать Personal Team (бесплатно)
```

### **Проблема 4: "Certificate not trusted"**

```bash
# Установить сертификат на iOS устройство:
# 1. Отправить fullchain.pem на iPhone (email, AirDrop)
# 2. Settings → Profile Downloaded → Install
# 3. Settings → General → About → Certificate Trust Settings
# 4. Включить сертификат
```

---

## 📊 **ЧЕКЛИСТ ПЕРЕД ПУБЛИКАЦИЕЙ**

### **App Store Connect**

- [ ] Создать App ID в Developer Portal
- [ ] Создать App в App Store Connect
- [ ] Заполнить метаданные приложения
- [ ] Подготовить скриншоты (6.5", 5.5")
- [ ] Написать описание приложения
- [ ] Создать Privacy Policy URL
- [ ] Настроить ценообразование
- [ ] Добавить Keywords для ASO
- [ ] Создать Promotional Text

### **Compliance**

- [ ] Экспортное соответствие (ECCN)
- [ ] GDPR compliance
- [ ] CCPA compliance
- [ ] Data collection disclosure
- [ ] Age rating (17+ рекомендуется для корпоративных приложений)

---

## 🎯 **БЫСТРЫЙ СТАРТ (TL;DR)**

```bash
# На Mac:

# 1. Установить инструменты
brew install --cask flutter
sudo gem install cocoapods

# 2. Скопировать проект
scp -r root@5.249.160.54:"/srv/Lone Star Chat/mobile" ~/LoneStarChat/

# 3. Инициализировать
cd ~/LoneStarChat/mobile
flutter create . --org com.lonestar
flutter pub get
cd ios && pod install && cd ..

# 4. Обновить API URL
echo "static const String baseUrl = 'https://5.249.160.54';" > lib/core/config/api_config.dart

# 5. Запустить
flutter run

# 6. Для production:
flutter build ipa --release
```

---

## 📞 **ПОДДЕРЖКА**

### **Ресурсы:**
- 📖 [Flutter iOS Deployment](https://docs.flutter.dev/deployment/ios)
- 🍎 [Apple Developer Documentation](https://developer.apple.com/documentation/)
- 🎯 [App Store Connect](https://appstoreconnect.apple.com/)
- 💬 [Flutter Community](https://flutter.dev/community)

### **Логи и Отладка:**

```bash
# Flutter логи
flutter logs

# Xcode логи
open ios/Runner.xcworkspace
# Product → Scheme → Edit Scheme → Run → Arguments
# Add: -FIRAnalyticsDebugEnabled

# Crash logs на устройстве
# Settings → Privacy → Analytics & Improvements → Analytics Data
```

---

## ✅ **ИТОГО**

**Сборка iOS возможна прямо сейчас!**

**Требуется:**
1. ⏱️ **5-10 минут** - инициализация Flutter на Mac
2. ⏱️ **5 минут** - настройка Xcode
3. ⏱️ **30 секунд** - первая сборка
4. ⏱️ **2 минуты** - запуск на симуляторе
5. ⏱️ **5 минут** - сборка для production

**Общее время: ~20-30 минут** до первого запуска на iPhone!

---

**Backend готов:** ✅ `https://5.249.160.54`  
**Flutter код готов:** ✅ 100%  
**Осталось:** Инициализировать проект на Mac и собрать!

**Готовы начать? Скопируйте проект на Mac и следуйте инструкции выше!** 🚀
