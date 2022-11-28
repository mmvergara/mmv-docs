---
sidebar_position: 2
---

# Auth

Authentication is handled by **[Express Session](http://expressjs.com/en/resources/middleware/session.html)** and the session storage is handled by mongodDB.

## Register

### Front-End

Client side validation as we know is just as important as server side. for this **[Formik](https://formik.org/)** with **[Yup](https://github.com/jquense/yup)**

```jsx
const formik = useFormik({
  initialValues: {
    email: "",
    username: "",
    password: "",
    confirmPassword: "",
  },
  validationSchema: registerValidationSchema,
  onSubmit: registerSubmitHandler,
});
```

### Back-End

On the backend, user input validation is handled by **[Express Validator](https://express-validator.github.io/docs/)** <br/>
Different Types of validation is done in this controller

- User Input Validation
- If Email already exists
- If Username already exists

```jsx
exports.putRegister = async (req, res, next) => {
  const validationErrors = validationResult(req);
  const { email, username, password } = req.body;

  try {
    if (email.includes(" ") || username.includes(" ") || password.includes(" ")) {
      throw newError("Spaces are not allowed in all fields", 422);
    }
    const newUser = new userModel({
      email,
      password: await bcrypt.hash(password, 8),
      username,
      messages: [],
      posts: [],
    });
    if (!validationErrors.isEmpty()) {
      let data = "";
      validationErrors.array().forEach((x) => (data += x.msg + "\n"));
      throw newError(`Registration Unsuccessful + ${data}`, 422);
    }
    const similarUsernameExist = userModel.findOne({ username: username });
    const similarEmailExist = userModel.findOne({ email: email });
    const foundSimilarities = await Promise.all([similarUsernameExist, similarEmailExist]);
    if (foundSimilarities[1] !== null) throw newError("Email already exists", 400);
    if (foundSimilarities[0] !== null) throw newError("Username already exists", 400);

    //Saving
    await newUser.save();
    res.status(201).send({
      message: "Registration Successful",
      ok: true,
    });
  } catch (err) {
    console.log("Something went wrong on authController putRegister");
    next(err);
  }
};
```

---

## Login

### Front-End

Form Validation is handled by **[Formik](https://formik.org/)** with **[Yup](https://github.com/jquense/yup)** as usual.<br/>

```jsx
const formik = useFormik({
  initialValues: {
    email: "",
    password: "",
  },
  validationSchema: loginValidationSchema,
  onSubmit: loginFormSubmitHandler,
});
```

Upon pressing successful login, user information is stored redux by dispatching an action

```jsx
authLogin (state: AuthState, action: PayloadAction<AuthState>) => {
  state.userpic = API_URL + "/" + action.payload.userpic;
  state.username = action.payload.username;
  state.isAuthenticated = action.payload.isAuthenticated;
  state.tokenExpirationDate = action.payload.tokenExpirationDate;
  localStorage.setItem(
    "authState",
    JSON.stringify({ ...action.payload, userpic: state.userpic })
  );
}
```

### Back-End

Validation

- User Input validation
- Invalid Email
- Invalid password

```jsx
exports.postLogin = async (req, res, next) => {
  const { email, password } = req.body;
  const validationErrors = validationResult(req);
  //Validation
  try {
    if (!validationErrors.isEmpty()) {
      let data = "";
      validationErrors.array().forEach((x) => (data += x.msg + "\n"));
      res.status(422).send({ message: "Login Unsuccessful", data });
    }

    if (email.includes(" ") || password.includes(" ")) {
      throw newError("Spaces are not allowed in all fields", 422);
    }

    const foundUser = await userModel.findOne({ email: email });
    if (!foundUser) throw newError("Invalid Email", 422);

    const comparisonResult = await bcrypt.compare(password, foundUser.password);
    if (!comparisonResult) throw newError("Invalid Password", 422);

    req.session.isLoggedIn = true;
    req.session.userId = foundUser._id;
    res.status(200).send({
      message: "Login Successful",
      userData: {
        username: foundUser.username,
        userpic: foundUser.userpic,
        isAuthenticated: true,
        tokenExpirationDate: new Date(req.session.cookie._expires).getTime(),
      },
      ok: true,
    });
    return;
  } catch (err) {
    console.log("Something went wrong on authController postLogin");
    next(err);
  }
};
```

After all of the validation the server will send a http only cookie for the session auth

---

## Logout

### Front-End

Clicking the logout button will trigger a request in the Back-End and reset redux state, even without waiting for the backend response

```jsx
  authLogout: (state: AuthState) => {
    postLogout(); //backend logout request
    state.userpic = "";
    state.username = "";
    state.isAuthenticated = false;
    state.tokenExpirationDate = 0;
    localStorage.removeItem("authState");
  },
```

### Back-End

`postLogout` a controller that will destroy the user session on the database.

```jsx
exports.postLogout = (req, res, next) => {
  req.session.destroy(function () {
    res.status(200).send({ message: "Logged Out Successfully", ok: true });
  });
};
```

note: an error can still occur here but instead we will ignore that and respond with a successful request anyways.

---

## Patch User Information

### Change UserPic

#### Front-End

```jsx
export const patchChangeUserPic = async (ChangeUserPicData: FormData): Promise<authResponse> => {
  const result = await AuthRequest({
    method: "patch",
    url: "/auth/patch/userpic",
    data: ChangeUserPicData,
    headers: {
      "Content-Type": "multipart/form-data",
    },
  });
  return result;
};
```

#### Back-End

```jsx
// VALIDATIONS ⬆⬆⬆
const foundUser = await userModel.findOne({ _id: userId });
await clearImage(foundUser.userpic);
try {
  if (!foundUser) throw newError("User not found", 422);
  foundUser.userpic = imgPath;
  await foundUser.save();
  res.status(200).send({ message: "Success: Image Uploaded", ok: true, imgPath });
} catch (error) {
  console.log('Something wen"t wrong in patchChangeEmail');
  next(error);
}
```

clearImage : clears image in the file system.

```jsx
const fs = require("fs");
const path = require("path");

module.exports = async (filepath) => {
  const imgPath = path.join(__dirname + "\\..\\" + filepath);
  fs.promises.unlink(imgPath).catch((err) => {});
};
```

---

### Change Email

#### Front-End

```jsx
export const patchChangeEmail = async (ChangeEmailData: ChangeEmailForm): Promise<authResponse> => {
  const result = await AuthRequest({
    method: "patch",
    url: "/auth/patch/email",
    data: ChangeEmailData,
  });
  return result;
};
```

#### Back-End

```jsx
// VALIDATIONS ⬆⬆⬆
const findUser = userModel.findOne({ _id: userId });
const findSameEmail = userModel.findOne({ email: newEmail });
const [foundUser, foundSameEmail] = await Promise.all([findUser, findSameEmail]);
try {
  if (foundSameEmail) throw newError("Someone is already using that Email", 422);
  if (!foundUser) throw newError("User not found", 422);

  const comparisonResult = await bcrypt.compare(password, foundUser.password);
  if (!comparisonResult) throw newError("Wrong Password");

  foundUser.email = newEmail;
  await foundUser.save();
  res.status(200).send({ message: "Success: Email Changed Successfully", ok: true });
} catch (err) {
  console.log('Something wen"t wrong in patchChangeEmail');
  err.statusCode = err.statusCode || 500;
  next(err);
}
```

---

### Change Username

#### Front-End

```jsx
export const patchChangeUsername = async (
  ChangeUsernameData: ChangeUsernameForm
): Promise<authResponse> => {
  const result = await AuthRequest({
    method: "patch",
    url: "/auth/patch/username",
    data: ChangeUsernameData,
  });
  return result;
};
```

#### Back-End

```jsx
// VALIDATIONS ⬆⬆⬆
const findUser = userModel.findOne({ _id: userId });
const findSameUserName = userModel.findOne({ username: newUsername });
const [foundUser, foundSameUsername] = await Promise.all([findUser, findSameUserName]);

try {
  if (foundSameUsername) throw newError("Someone is already using that username", 422);
  if (!foundUser) throw newError("User not found", 422);

  const comparisonResult = await bcrypt.compare(password, foundUser.password);
  if (!comparisonResult) throw newError("Wrong Password");

  foundUser.username = newUsername;
  await foundUser.save();
  res.status(200).send({ message: "Success: Username Changed Successfully", ok: true });
} catch (err) {
  console.log('Something wen"t wrong in patchChangeUsername');
  err.statusCode = err.statusCode || 500;
  next(err);
}
```

### Change Password

#### Front-End

```jsx
export const patchChangePassword = async (
  ChangePassData: ChangePassForm
): Promise<authResponse> => {
  const result = await AuthRequest({
    method: "patch",
    url: "/auth/patch/password",
    data: ChangePassData,
  });
  return result;
};
```

#### Back-End

```jsx
const foundUser = await userModel.findOne({ _id: userId });
try {
  if (!foundUser) throw newError("User not found", 422);

  const comparisonResult = await bcrypt.compare(oldPassword, foundUser.password);
  if (!comparisonResult) throw newError("Wrong Password");

  foundUser.password = await bcrypt.hash(newPassword, 8);
  await foundUser.save();
  res.status(200).send({ message: "Password Changed Successfully", ok: true });
} catch (err) {
  console.log('Something wen"t wrong in patchChangePassowrd');
  next(err);
}
```
