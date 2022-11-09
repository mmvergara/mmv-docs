---
sidebar_position: 4
---

# Notes

Ashera Notes are just text documents with title and a content.
We are using `onSnapshot` to update the UI everytime a change in the database is made.

```jsx
  useEffect(() => {
    const curUserId = auth?.uid;
    if (!curUserId) return;
    const q = query(notesRef, where("noteAuthor", "==", curUserId));
    onSnapshot(q, (snapshot) => {
      const notes: noteDetails[] = [];
      snapshot.docs.forEach((n) => {
        const note = n.data() as noteDetails;
        notes.push({ ...note, noteId: n.id });
      });
      setAllNotes(notes);
      setIsLoading(false);
    });
  }, [auth]);

```

## Creating Notes
Creating Notes is straightforward, the user needs to be signed in and we are using the `addDoc` firestore method to do operations
```jsx
export const addNote = async (noteDetails: noteDetail) => {
  const author = authFB.currentUser?.uid!;
  if (!author) return;
  const noteInfo: noteNoId = { ...noteDetails, noteAuthor: author };
  const result = await addDoc(notesRef, noteInfo);
  return result;
};
```

## Deleting Notes
Deleting notes only needs a noteId and using the Cloud Firestore Security Rules only users who owns that note can delete it.
```jsx
export const deleteNote = async (noteId: string) => {
  const result = await deleteDoc(doc(notesRef, noteId));
  return result;
};
```

## Editing Notes

```jsx
export const editNote = async (noteDetails: noteDetail, noteId: string) => {
  const author = authFB.currentUser?.uid!;
  if (!author) return;
  const noteInfo: noteNoId = { ...noteDetails, noteAuthor: author };
  const result = await updateDoc(doc(firestoreDB, "Notes", noteId), { ...noteInfo });
  return result;
};
```
