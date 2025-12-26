# 🌍 Translation Plan: Russian → English (Alpha 0.26)

## ✅ Pre-Translation Checklist

- [x] Alpha 0.25 saved to git (commit: be1b9eb, tag: alpha-0.25)
- [x] ZIP backup created (3.8 GB)
- [x] Restore script ready (RESTORE_ALPHA_0.25.sh)
- [x] Version history documented
- [ ] Translation in progress...

---

## 📋 Files to Translate

### 🎯 Priority 1: Mobile App (Dart Files)

User-facing strings that need translation:

1. **Authentication**
   - [ ] `lib/screens/auth/login_screen.dart`
   - [ ] `lib/providers/auth_provider.dart`
   - [ ] `lib/services/auth_service.dart`

2. **Main Screens**
   - [ ] `lib/screens/home/home_screen.dart`
   - [ ] `lib/screens/splash/splash_screen.dart`
   - [ ] `lib/screens/force_update/force_update_screen.dart`

3. **Tabs**
   - [ ] `lib/screens/home/tabs/chats_tab_screen.dart`
   - [ ] `lib/screens/home/tabs/employees_screen.dart`
   - [ ] `lib/screens/home/tabs/team_chats_screen.dart`
   - [ ] `lib/screens/home/tabs/voice_rooms_screen.dart`

4. **Other Screens**
   - [ ] `lib/screens/admin/admin_panel_screen.dart`
   - [ ] `lib/screens/profile/profile_screen.dart`
   - [ ] `lib/screens/team/team_chat_screen.dart`
   - [ ] `lib/screens/team/voice_room_screen.dart`
   - [ ] `lib/screens/ai_chat/ai_chat_screen.dart`

5. **Services & Core**
   - [ ] `lib/services/api_service.dart`
   - [ ] `lib/services/version_service.dart`
   - [ ] `lib/main.dart`
   - [ ] `lib/app.dart`

6. **Models**
   - [ ] `lib/models/channel.dart`
   - [ ] `lib/models/user.dart`
   - [ ] `lib/models/message.dart`

7. **Utils**
   - [ ] `lib/utils/url_utils.dart`

---

### 🎯 Priority 2: Documentation (MD Files)

Keep Russian versions, create English versions:

- [ ] README.md → README_EN.md
- [ ] QUICKSTART.md → QUICKSTART_EN.md  
- [ ] PROJECT_SUMMARY.md → PROJECT_SUMMARY_EN.md
- [ ] IOS_BUILD_GUIDE.md → IOS_BUILD_GUIDE_EN.md
- [ ] TESTING_GUIDE.md → TESTING_GUIDE_EN.md
- [ ] DEPLOYMENT_CHECKLIST.md → DEPLOYMENT_CHECKLIST_EN.md

---

### 🎯 Priority 3: Scripts & Comments

- [ ] `mobile/fix_pods.sh` - comments
- [ ] `mobile/RADICAL_FIX_PODS.sh` - output messages
- [ ] `backend/server-chat.js` - console.log messages
- [ ] Other shell scripts

---

## 🔍 Translation Strategy

### What to Translate:
1. **UI Strings** - All user-visible text
   - Button labels
   - Screen titles
   - Error messages
   - Success messages
   - Placeholders
   - Help text

2. **Console Logs** (Debug messages)
   - Keep emoji prefixes (🔑, ✅, ❌, 📥, 📤, etc.)
   - Translate message text
   - Examples:
     - `'🔑 Токен добавлен'` → `'🔑 Token added'`
     - `'✅ Успешно загружено'` → `'✅ Successfully loaded'`

3. **Comments** (Code documentation)
   - Function descriptions
   - Complex logic explanations
   - TODO comments

### What NOT to Translate:
1. **Variable names** - Keep as is
2. **Function names** - Keep as is
3. **API endpoints** - Keep as is
4. **Database field names** - Keep as is
5. **File names** - Keep as is (create new English docs instead)
6. **Git commit messages** - Already in English

---

## 📝 Translation Examples

### Before:
```dart
Text('Сотрудники'),
print('🔑 Токен найден'),
// Проверяем авторизацию
```

### After:
```dart
Text('Employees'),
print('🔑 Token found'),
// Check authorization
```

---

## 🧪 Testing After Translation

### 1. Build Test
```bash
cd mobile
flutter clean
flutter pub get
flutter run -d Curtis
```

### 2. Functionality Test
- [ ] Login screen displays correctly
- [ ] All tabs show English text
- [ ] Error messages in English
- [ ] Console logs readable
- [ ] No broken UI elements

### 3. Verification
- [ ] No Russian text visible to user
- [ ] All buttons/labels in English
- [ ] Error handling works
- [ ] Navigation working

---

## 📦 After Translation

### Create Alpha 0.26
```bash
git add -A
git commit -m "Alpha 0.26 - Full English translation

- Translated all UI strings to English
- Translated console logs and debug messages
- Translated code comments
- Created English documentation
- Tested on iOS device
"

git tag alpha-0.26
```

### Backup Alpha 0.26
```bash
cd ~/Desktop
zip -r "Lone Star Chat Backups/LoneStarChat-Alpha-0.26-$(date +%Y%m%d).zip" \
  "Lone Star Chat" -x "*.git/*" "*/node_modules/*" "*/build/*" "*/Pods/*"
```

---

## 🎯 Success Criteria

Alpha 0.26 is complete when:
- [ ] No Russian text visible in mobile app UI
- [ ] All console logs in English
- [ ] All code comments in English
- [ ] English documentation created
- [ ] App builds successfully
- [ ] App runs without errors
- [ ] All functionality tested
- [ ] Git tag created
- [ ] Backup created

---

## ⚠️ Safety Net

If something breaks during translation:
```bash
./RESTORE_ALPHA_0.25.sh
```

Or:
```bash
git checkout alpha-0.25
```

**Alpha 0.25 is always available for restore!** ✅

---

**Ready to start translation!** 🚀
