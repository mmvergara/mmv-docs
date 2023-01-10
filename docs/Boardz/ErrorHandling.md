---
sidebar_position: 3
---

# Utitities

## User Input / Form Validation

### Front-End

Client side validation as we know is just as important as server side. for this **[Formik](https://formik.org/)** with **[Yup](https://github.com/jquense/yup)**
```jsx
import * as yup from "yup";
export const loginValidationSchema = yup.object({
  email: yup.string().email("Enter a valid email").required("Email is required."),
  password: yup
    .string()
    .min(8, "Minimum of 8 characters")
    .max(30, "Maximum of 30 characters")
    .trim()
    .required("Password field is required."),
});
```

```jsx
import * as yup from "yup";
  const formik = useFormik({
    initialValues: {
      email: "",
      password: "",
    },
    validationSchema: loginValidationSchema,
    onSubmit: loginFormSubmitHandler,
  });
```

### Back-End

On the backend, user input validation is handled by **[Express Validator](https://express-validator.github.io/docs/)** <br/>
Different Types of validation is done in this controller
```jsx
router.post(
  "/login",
  [
    body("email").trim().isEmail().normalizeEmail().withMessage("Email is Invalid"),
    body("password").trim().isLength({ min: 8, max: 30 }).withMessage("Password is Invalid"),
  ],
  postLogin
);
```
---
## Error Handling

Error Handling is handled by 1 helper function and 1 middleware to minimize the work
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
