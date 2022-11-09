---
sidebar_position: 2
---

# Auth

Auth is managed by **[Firebase Auth](https://firebase.google.com/products/auth)** system
Client side user input validation is handled by **[Formik](https://formik.org/)** and **[Yup](https://github.com/jquense/yup)**

## Sign up

Using the firebase `createUserWithEmailAndPassword` built in method to Sign up a user. <br/>
We also update the user `displayName` after registering the user.

```jsx
import { createUserWithEmailAndPassword, updateProfile } from "firebase/auth";
const authFB = getAuth();
```

```jsx
  const handleSignUp = async () => {
    let currentUser;
    try {
      const result = await createUserWithEmailAndPassword(
        authFB,
        email,
        password
      );
      currentUser = result;
      await updateProfile(currentUser.user, { displayName: username});
    } catch (error) {
      const err = error as FirebaseError;
      if (currentUser) {
        toast({
          description: "The Account was created but the username was not set",
          status: "warning",
          duration: 9000,
          isClosable: true,
        });
      } else {
       // set error
      }
    }
  };
```

## Sign in

Using the firebase `signInWithEmailAndPassword` built in method to Sign in the user

```jsx
import { signInWithEmailAndPassword } from "firebase/auth";
```

```jsx
const handleSignIn = async () => {
  setStatus({ hasError: false, isLoading: true, statusText: "" });
  try {
    const result = await signInWithEmailAndPassword(
      authFB,
      formik.values.SignInEmail,
      formik.values.SignInPassword
    );
    toast({
      description: "Login Successful",
      status: "success",
      duration: 2000,
      isClosable: true,
    });
    if (result) setStatus({ hasError: false, isLoading: false, statusText: "Logged In!" });
  } catch (error) {
    setStatus({ hasError: true, isLoading: false, statusText: "Invalid Credentials" });
  }
};
```

## Sign out

Using the firebase `signOut` built in method to Sign out the user

```jsx
import { signOut } from "firebase/auth";
```

```jsx
const signOutHandler = () => {
  toggleSignin(false);
  signOut(authFB);
  toast({
    description: "Logged Out",
    status: "error",
    duration: 2000,
    isClosable: true,
  });
  navigate("/"); //Go back to landing page
};
```

## Auth State

The Auth State is managed by React [Context API](https://reactjs.org/docs/context.html) and we are exporting a custom hook to get the current Auth State.

```jsx
import { User } from "firebase/auth";
import { createContext, useState, useEffect, useContext } from "react";
import { authFB } from "../firebase";

const AuthContext = createContext<User | null>(null);
export const useAuth = () => useContext(AuthContext);

export const AuthProvider = ({ children }: { children: JSX.Element | JSX.Element[] }) => {
  const [loading, setLoading] = useState<boolean>(false);
  const [user, setUser] = useState<User | null>(null!);
  useEffect(() => {
    authFB.onAuthStateChanged((user) => {
      if (user) {
        setUser(user);
        return;
      }
      setUser(user);
      setLoading(false);
    });
  }, [user]);

  return <AuthContext.Provider value={user}>{!loading && children}</AuthContext.Provider>;
};

```
