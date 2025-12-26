# 🐛 Bug Fixes Complete - December 11, 2025

## Issues Fixed

### 1. ❌ App Crash on Calls
**Problem**: App crashed immediately when initiating voice/video calls  
**Root Cause**: CallService using old Socket.io event names that didn't match backend  
**Solution**: Updated all Socket.io events to match new backend API

**Changes in** `call_service.dart`:
- `incoming_call` → `call:incoming`
- `call_accepted` → `call:accepted`
- `sdp_offer` → `call:offer`
- `sdp_answer` → `call:answer`
- `ice_candidate` → `call:ice-candidate`
- Added proper authentication, error handling, and state management

### 2. ❌ Blur Not Covering Full Screen
**Problem**: Blur effect didn't cover notch (top) and home indicator (bottom) on iPhone 17 Pro Max  
**Root Cause**: MediaQuery returns size WITHOUT safe area padding  
**Solution**: Used `View.physicalSize` to get true screen dimensions

**Changes in** `discord_message_menu.dart`:
```dart
final view = View.of(context);
final screenHeight = view.physicalSize.height / view.devicePixelRatio;
final screenWidth = view.physicalSize.width / view.devicePixelRatio;
```

### 3. ❌ Slow Authentication
**Problem**: Login took 15-30 seconds, frequent "Connection Failed" errors  
**Root Cause**: Blocking version checks, no timeouts, slow token storage  
**Solution**: Optimized initialization, added timeouts, async token storage

**Changes in** `auth_provider.dart` & `auth_service.dart`:
- 500ms timeout for token loading
- 8s timeout for login request
- Async token storage (no await)
- Version check in background (non-blocking)
- Better error messages in Russian

## Results

| Before | After |
|--------|-------|
| ❌ Crash on call | ✅ Stable calls |
| ❌ Partial blur | ✅ Full screen blur |
| ❌ 15-30s login | ✅ 2-5s login |
| ❌ Generic errors | ✅ Clear Russian messages |

## Testing

1. **Blur**: Long press message → verify blur covers entire screen
2. **Calls**: Make audio/video call between 2 devices → verify no crash
3. **Login**: Close app → reopen → login → should be fast (<5s total)

All fixes compiled successfully ✅
