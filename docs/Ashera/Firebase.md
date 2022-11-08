---
sidebar_position: 10
---

# Firebase

Ashera is using Firebase as a backend more specifically 'Firestore' for realtime updates

## Frontend Setup

```jsx
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { addDoc, arrayUnion, arrayRemove, collection, doc } from "firebase/firestore";
import { getFirestore, updateDoc, serverTimestamp, deleteDoc } from "firebase/firestore";

//Firebase App Initialize
export const appFB = initializeApp(firebaseConfig);

//Services
export const firestoreDB = getFirestore();
export const authFB = getAuth();

//References
export const taskSectionRef = collection(firestoreDB, "TaskSections");
export const notesRef = collection(firestoreDB, "Notes");
```

## Cloud Firestore Security rules

 - We allow read,update,delete if the user is authenticated and is the owner
 - We allow create if the user is authenticated

```javascript
  rules_version = '2';
  service cloud.firestore {
    match /databases/{database}/documents {

        match /TaskSections/{TaskSectionName} {
          allow read,update,delete: if request.auth != null && request.auth.uid == resource.data.author;
          allow create: if request.auth != null;
        }

        match /Notes/{TaskSectionName} {
          allow read,update,delete: if request.auth != null && request.auth.uid == resource.data.noteAuthor;
          allow create: if request.auth != null;
        }
        
    }
  }
```
