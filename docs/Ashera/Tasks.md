---
sidebar_position: 3
---

# Tasks

Ashera Tasks are organized in Task Sections, Each section has a name and tasks can be created inside of each section.


## Create Task Section / Task

Creating Task Section only needs a name and a user to be authenticated. Firebase really makes creating a doc really easy with their built in functions. `addDoc` from

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
};
```

```jsx

```

## Create Task Section / Task

```jsx

```

```jsx

```
