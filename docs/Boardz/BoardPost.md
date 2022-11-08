---
sidebar_position: 6
---

# Board Post

A Board Post is basically just the same thing as a "Post" on any social media app. like a post in facebook or a tweet in twitter

## Creating a Post

Creating a post only needs a Title and a content

```jsx
export const putCreateBoardz = async (
  boardData: newBoardPostData
): Promise<newBoardPostDataResponse> => {
  const result = (await AxiosRequest({
    method: "put",
    url: "/boardz/createnewboardz",
    data: boardData,
  })) as AxiosResponse<newBoardPostDataResponse>;
  return result.data;
};

```

Backend

```jsx
// VALIDATIONS ⬆⬆⬆
try {
  const foundUser = await userModel.findOne({ _id: userId });
  if (!foundUser) throw newError("Invalid User", 401);

  const savePostResult = await newBoardPost.save();
  foundUser.posts.push(savePostResult._id);
  await foundUser.save();
  res
    .status(200)
    .send({ savePostResult: savePostResult, ok: true, message: "Board Created Succesfully" });
} catch (err) {
  console.log("Something went wrong on boardzController in putCreateBoardz");
  next(err);
}
```

---

## Likes

Liking doesn't use websockets for realtime updates instead it triggers a new request upon liking or disliking to update the like count

```jsx
const updateHandler = () => updateBoardz(boardId);

const LikeCommentHandler = async (commentId: string) => {
  await postLikeCommentHandler(commentId);
  updateHandler();
};
```

## Comments

Like 'Likes' same thing happens when a comment is submitted by the user it sends a new request so all of the likes and comments are seen

```jsx
const updateHandler = () => updateBoardz(boardId);

const submitCommentHandler = async () => {
  if (formik.values.commentContent.trim().length === 0) return;
  const result = await putCommentHandler(boardId, formik.values.commentContent);
  if (!result.ok) return;
  submitComment(); // UpdateHandler Prop
  formik.resetForm();
};
```
