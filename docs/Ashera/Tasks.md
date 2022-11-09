---
sidebar_position: 3
---

# Tasks

Ashera Tasks are organized in Task Sections, Each section has a name and tasks can be created inside of each section.
We are using `onSnapshot` to update the UI everytime a change in the database is made.

```jsx
  useEffect(() => {
    const curUserId = auth?.uid;
    if (!curUserId) return;
    const q = query(taskSectionRef, where("author", "==", curUserId));
    onSnapshot(q, (snapshot) => {
      const sections: TS[] = [];
      snapshot.docs.forEach((s) => {
        const section = s.data() as TS;
        sections.push({ ...section, id: s.id });
      });
      setIsLoading(false);
      setAllTaskSection(sections.sort((a, b) => a.createdAt.seconds - b.createdAt.seconds));
    });
  }, [auth]);
```

## Creating and Deleting Task Section

Creating Task Section only needs a name and a user to be authenticated. Firebase really makes creating and deleting a doc really easy with their built in functions `addDoc` and `deleteDoc` from firestore.

```jsx
//Refer to firebase section for imports and setups
export const addNewTaskSection = async (taskSectionName: string) => {
  const userId = authFB.currentUser?.uid!;
  if (!userId) return;
  await addDoc(taskSectionRef, {
    author: userId,
    taskSectionName,
    tasks: [],
    createdAt: serverTimestamp(),
  });

export const deleteTaskSection = async (sectionId: string) => {
  const userId = authFB.currentUser?.uid!;
  if (!userId) return;
  return await deleteDoc(doc(taskSectionRef, sectionId));
};
};
```

## Creating and Deleting Task

Creating and Deleting a Task is basically just the same as Creating and Deleting Task Section

```jsx
//Refer to firebase section for imports and setups
export const addTask = async (taskSectionId: string, taskName: string) => {
  const userId = authFB.currentUser?.uid!;
  if (!userId) return;
  return await updateDoc(doc(firestoreDB, "TaskSections", taskSectionId), {
    tasks: arrayUnion(taskName),
  });
};

export const deleteTask = async (taskSectionId: string, taskName: string) => {
  return await updateDoc(doc(firestoreDB, "TaskSections", taskSectionId), {
    tasks: arrayRemove(taskName),
  });
};
```
