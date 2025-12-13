# Firestore Security Rules Setup

## Problem
If you're seeing "Error syncing tasks from database", this is caused by Firestore security rules blocking access.

## Solution

### Step 1: Access Firebase Console
1. Go to https://console.firebase.google.com/
2. Select your project: **task-1c051**

### Step 2: Update Firestore Security Rules
1. Click on **Firestore Database** in the left sidebar
2. Click on the **Rules** tab
3. Replace the existing rules with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow authenticated users to access their own tasks
    match /artifacts/{appId}/users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

4. Click **Publish** to save the rules

### Step 3: Verify the Fix
1. Refresh your Task Tracker application
2. Sign in with Google
3. Try adding a task
4. The error should be gone!

## Alternative: Temporary Development Rules (NOT for production!)

If you just want to test quickly, you can use these permissive rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

**⚠️ WARNING:** These rules allow any authenticated user to read/write ALL documents. Only use for development!

## Debugging

To see the exact error:

1. Open browser DevTools (F12)
2. Go to the **Console** tab
3. Look for error messages showing:
   - Error code (e.g., "permission-denied")
   - Error message
   - Stack trace

## Common Error Codes

- **`permission-denied`**: Security rules are blocking access → Follow steps above
- **`not-found`**: Collection doesn't exist → Will be created automatically when you add first task
- **`unauthenticated`**: Not signed in → Click "Sign In with Google"
- **`unavailable`**: Network issues → Check internet connection

## Production-Ready Rules

For production deployment, use these secure rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only access their own data
    match /artifacts/{appId}/users/{userId}/tasks/{taskId} {
      allow read, write: if request.auth != null
                          && request.auth.uid == userId;
    }

    // Allow listing tasks
    match /artifacts/{appId}/users/{userId}/tasks {
      allow list: if request.auth != null
                    && request.auth.uid == userId;
    }

    // Validate task data
    match /artifacts/{appId}/users/{userId}/tasks/{taskId} {
      allow create: if request.auth != null
                    && request.auth.uid == userId
                    && request.resource.data.title is string
                    && request.resource.data.quadrant is number
                    && request.resource.data.quadrant >= 1
                    && request.resource.data.quadrant <= 4;

      allow update: if request.auth != null
                    && request.auth.uid == userId;
    }
  }
}
```

## Need More Help?

Check the browser console for detailed error messages, or visit:
- Firebase Docs: https://firebase.google.com/docs/firestore/security/get-started
- Firebase Support: https://firebase.google.com/support
