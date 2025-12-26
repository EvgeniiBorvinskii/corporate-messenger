# 🧪 Testing Guide - Admin Impersonation

## Quick Start

### 1. Login as Master Account
```
Email: admin@lonestar.local
Password: admin123
```

### 2. Navigate to Profile Tab
- Bottom navigation → Profile (last tab)
- Scroll down to see "Master Account" section with ⭐ icon
- Find orange button: "View as User (Test Roles)"

### 3. Test Impersonation

#### Test Case 1: View as Sales User
1. Tap "View as User (Test Roles)"
2. Search: "Simon"
3. Select "Simon Clarke" (General Sales Manager)
4. Confirm dialog → "View as User"
5. **Expected Result:**
   - 🔴 Red banner appears: "Admin View Mode - Viewing as: Simon Clarke"
   - Profile shows Simon's info
   - Chats tab shows only Sales channels
6. Tap "Exit View" in red banner
7. **Expected Result:**
   - Banner disappears
   - Back to Master account

#### Test Case 2: View as Service User
1. Tap "View as User (Test Roles)"
2. Search: "Tommy"
3. Select "Tommy Ton" (Service Advisor)
4. Confirm → "View as User"
5. **Expected Result:**
   - Red banner: "Viewing as: Tommy Ton"
   - Service channels visible
   - Service team chats accessible
6. Exit View

#### Test Case 3: Quick Switch Between Users
1. Enter impersonation mode (any user)
2. Tap Profile tab
3. Tap "View as User (Test Roles)" again
4. Select different user
5. **Expected Result:**
   - Smoothly switches to new user
   - Red banner updates name
   - No logout required

#### Test Case 4: Search Functionality
1. Open "View as User" screen
2. Try searches:
   - "Sales" → Shows all Sales department
   - "@lonestar" → Shows all with email
   - "Manager" → Shows users with Manager in position
3. Clear search (X button)
4. **Expected Result:**
   - Instant filtering
   - Shows relevant users only
   - Clear returns full list

#### Test Case 5: Banner Persistence
1. Enter impersonation as any user
2. Navigate between tabs:
   - Home → Chats → Team → Voice → Schedule → Employees → Profile
3. **Expected Result:**
   - 🔴 Red banner always visible at top
   - "Exit View" always accessible
   - User name always shown

#### Test Case 6: Permissions Testing
1. Impersonate "Sales" role user
2. Check Chats tab → Should see Sales channels
3. Check Team tab → Should see Sales team chat
4. Exit View
5. Impersonate "Service" role user
6. Check Chats tab → Should see Service channels
7. **Expected Result:**
   - Different roles see different channels
   - Access restricted correctly

---

## 🐛 What to Look For

### ✅ Good Signs:
- Red banner appears immediately after impersonation
- User name in banner matches selected user
- Exit View button works from anywhere
- No lag when switching users
- Profile shows impersonated user's data
- Snackbar confirms "Now viewing as [Name]"
- After exit: "Returned to Admin account"

### ❌ Potential Issues:
- Banner doesn't appear
- Wrong user name shown
- Exit View doesn't work
- App crashes on user switch
- Channels don't update after impersonation
- Can't access impersonation screen

---

## 🔍 Debug Checklist

If something doesn't work:

1. **Check server logs:**
```bash
cd "/Users/svetanaborvinskaia/Desktop/Lone Star Chat"
tail -50 logs/backend-stdout.log | grep -i impersonate
```

2. **Check Flutter logs:**
- Look for "🎭 Starting impersonation..."
- Look for "✅ Impersonation started: [Name]"
- Look for errors in console

3. **Verify Master account:**
- Only ID: 1 (admin) can impersonate
- Check roles include "master"

4. **Test backend directly:**
```bash
# Get token after login
TOKEN="your_token_here"

# Start impersonation
curl -X POST http://192.168.3.213:3000/api/admin/impersonate/1002 \
  -H "Authorization: Bearer $TOKEN"

# Stop impersonation
curl -X POST http://192.168.3.213:3000/api/admin/stop-impersonation \
  -H "Authorization: Bearer $TOKEN"
```

---

## 📊 Expected API Responses

### Start Impersonation:
```json
{
  "message": "Impersonation started",
  "user": {
    "id": "1002",
    "full_name": "Simon Clarke",
    "email": "simon_clarke@lonestar.local",
    "roles": ["sales"],
    "department": "Sales"
  },
  "isImpersonating": true,
  "realUserId": "1",
  "realUserName": "Lone Star Admin"
}
```

### Stop Impersonation:
```json
{
  "message": "Impersonation stopped",
  "user": {
    "id": "1",
    "full_name": "Lone Star Admin",
    "roles": ["master"]
  },
  "isImpersonating": false
}
```

---

## 🎯 Success Criteria

All these should work:
- ✅ Enter impersonation mode
- ✅ See red banner with user name
- ✅ Exit impersonation mode
- ✅ Switch between multiple users
- ✅ Search for users by name/email/dept
- ✅ See correct channels for each role
- ✅ Banner persists across all tabs
- ✅ No crashes or errors
- ✅ Smooth transitions
- ✅ Clear visual feedback (snackbars)

---

## 📱 User Experience Flow

```
Master Account (Admin)
        ↓
Profile → "View as User" button
        ↓
List of 35 employees
        ↓
Search/Select User → Confirm
        ↓
🔴 RED BANNER APPEARS
        ↓
View app as selected user
        ↓
Test channels/permissions
        ↓
"Exit View" button
        ↓
Back to Master Account
```

---

## ✅ Testing Complete When:

1. You can impersonate any of 35 users
2. Red banner always shows when in mode
3. Exit View works instantly
4. Channels change based on role
5. Search works correctly
6. No errors in logs
7. Smooth user experience

**Happy Testing!** 🚀
