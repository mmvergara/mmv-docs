---
sidebar_position: 3
---

# Error Handling

Error Handling is handled by 1 helper function and 1 middleware to minimize the work

## Error Handling Helper Function

This function returns an Error Object it is just so we can easily pass an `errorMessage` and `errorCode` rather than manually setting it after creating a new Error object.

```jsx
module.exports = (errorMessage, errorCode) => {
  const newErr = new Error(errorMessage);
  newErr.statusCode = errorCode || 500;
  return newErr;
};
```

### Usage

This is how i normally handle error and pass an error code

```jsx
const foundUser = await userModel.findOne({ email: email });
if (!foundUser) {
  const newErr = new Error("Invalid Email");
  newErr.statusCode = 422;
  throw newErr;
}
```

With the Helper Function

```jsx
const foundUser = await userModel.findOne({ email: email });
if (!foundUser) throw newError("Invalid Email", 422);
```

it makes handling error and attach a statusCode easy
