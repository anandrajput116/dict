# Shabd LexiView Learning Hub

## What changed
The Learning Hub is an added section in the existing single-file Shabd LexiView app. It keeps the original dictionary, decks, voice-and-type cards, portable HTML export, and IndexedDB storage. Learning cards are stored in the same `flashcard_deck` IndexedDB object store, so the existing spaced repetition queue can study them.

## Review scheduler

- Successful recall advances through **1, 3, 7, 21, 45, 90, 180, and 365 days**. After 365 days, successful recalls remain on annual reviews.
- An incorrect, difficult, or partly recalled answer starts a recovery run at **10 minutes**. Successful recovery recalls then schedule **1 hour, 4 hours, 1 day, and 2 days**. Once the 2-day recovery review succeeds, the card returns to the standard sequence at 1 day.
- Each attempt records its timestamp, grade, response time, confidence, difficulty estimate, stage, and interval. Card data and review history are saved in IndexedDB after each answer.
- Difficulty is derived from the card's error count and accuracy. Recall probability is an estimate that decays with elapsed time relative to the scheduled interval; it is a prioritization aid, not a validated clinical or psychometric measurement.
- Mastery requires multiple successful recalls and longer intervals: 5 successful attempts at stage 4 for Mastered, or 7 at stage 6 for Long-term retention.

## Firebase cloud sync setup

The provided Firebase web app config is preloaded. The app connects and syncs on startup. Google sign-in requires serving the app over HTTPS rather than opening the HTML file directly. This output folder includes Firebase Hosting config. With Firebase CLI installed and authenticated, run `firebase deploy --only hosting` from this folder, then open `https://anand-english-dictionary.web.app/dictionary.html`.

In Firebase Console, also:

1. Enable **Anonymous** and **Google** under Firebase Authentication providers. Firebase Hosting's default domain is used for sign-in; if you use a different domain, add it under Authorized domains.
2. Create a Cloud Firestore database.
3. Publish Firestore rules that require authentication and scope reads and writes to the active user's own `users/{uid}/...` documents. Example starting point:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

Google sign-in upgrades the initial anonymous Firebase profile when possible so local progress keeps the same UID. Google verifies the Gmail address; Firebase does not send a separate numeric email code for this Google flow. A numeric OTP requires a trusted Cloud Function and email delivery provider, neither included here. Firebase web config is public client configuration; never paste service account credentials into the app.

## Data shape

Firestore stores cards in `users/{uid}/cards`, voice/type cards in `users/{uid}/vtCards`, dictionary pages in `users/{uid}/pages`, and decks, bookmarks, theme, and page settings in `users/{uid}/profile/settings`. Large HTML content is split into subcollection chunks to fit Firestore document limits. The bundled built-in dictionary remains in the HTML; uploaded or edited pages sync per account.

Each learning card keeps the existing card fields (`id`, `title`, `htmlContent`, `stage`, `intervalDays`, `nextDueDate`, and deck IDs) and adds `front`, `answerText`, `category`, `questionType`, `reviewHistory`, `attemptCount`, `forgottenCount`, `recoveryActive`, and `recoveryStep`. Every review-history entry contains `at`, `grade`, `responseMs`, `confidence`, `difficulty`, `stageBefore`, and `intervalBefore`.

Use the Hub's Export button for a JSON backup. Import merges cards by ID into the local collection.






