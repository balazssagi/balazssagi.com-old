---
title: "Simplify React Logic with State Enums"
date: 2020-05-26
layout: layouts/post.njk
---

### The Problem

Imagine you're building the following UI flow:

![A UI flow with four screens connected by arrows. On the first screen there's an email input field a submit button, and an arrow to the second screen. On the second screen there's a loading indicator. From the second screen there're two arrows: one goes to the third screen with a "Thank you" message, and the other goes to the fourth screen, with and email input field, a "Retry" button and a "Something went wrong" error message. There's an arrow going to the second screen from here.](/img/ui-flow.png)

There are four different states. First, there’s a form with a submit button. When clicking on the button, a request is sent to the server, and the component displays a loading indicator. After that, it either shows a “Thank you” message or—in case of an error—shows the form again with a “Retry” button and an error message.

One way of dealing with these various states is using multple boolean state variables. It's pretty common to see something like this in React codebases:

```js
function App() {
    const [isLoading, setIsLoading] = React.useState(false);
    const [hasError, setHasError] = React.useState(false);
    const [isSuccess, setIsSuccess] = React.useState(false);

    // Rest of the component...
}
```

<a href="https://codesandbox.io/s/quizzical-water-4vu7w" target="_blank">See the full implementation on Codesandbox.</a>

It kinda works, but has some drawbacks. The main one is that it allows impossible states. There are three booleans so our component can be in the following 2<sup>3</sup> = 8 states:

<div class="table-wrap">

| `isLoading` | `hasError`  | `isSuccess` | note                                                                        |
| ----------- | ----------- | ----------- | --------------------------------------------------------------------------- |
| `false`     | `false`     | `false`     | Waiting for user submitting the form                                        |
| `true`      | `false`     | `false`     | Display the loading indicator                                               |
| `false`     | `false`     | `true`      | Success! Display the thank you message.                                     |
| `false`     | `true`      | `false`     | Oops, maybe the server is down? Display an error message and a retry button |
| **`false`** | **`true`**  | **`true`**  | **???**                                                                     |
| **`true`**  | **`false`** | **`true`**  | **???**                                                                     |
| **`true`**  | **`true`**  | **`false`** | **???**                                                                     |
| **`true`**  | **`true`**  | **`true`**  | **???**                                                                     |

</div>

The last four rows in the table aren’t meaningful in the context of this UI. Should we display the thank you message or the error message when both `hasError` and `isSuccess` are `true`? It’s easy to mess things up when juggling with multiple booleans, and it only gets worse with a more complex component. We have to be extra careful, so we don't set the component to any of the invalid states, because it could lead to all kinds of bugs.

Conditional rendering would look something like this with the boolean flags approach:

```jsx
function App() {
    const [isLoading, setIsLoading] = React.useState(false);
    const [hasError, setHasError] = React.useState(false);
    const [isSuccess, setIsSuccess] = React.useState(false);

    if (isLoading) {
        return <Loader />;
    }

    if (isSuccess) {
        return <ThankYouMessage />;
    }

    if (hasError) {
        return <FormWithErrorMessage />;
    }

    return <Form />;
}
```

Everything goes well until the component somehow ends up in an invalid state (for example there’s bug somewhere in your fetching logic). When both `hasError` and `isSuccess` are `true`, the user will se see the thank you message, because of the early return in the `isSuccess` branch.

### A Solution

You could ditch the three state variables, and use a single one with a string value:

```js
function Form() {
    // "idle" | "loading" | "error" | "success"
    const [status, setStatus] = React.useState("idle");

    // Rest of the component...
}
```

Our component’s four possible states will be represented by a `status` state variable. It can be either `idle`, `loading`, `error` or `success`. Typing these is quite error prone so I recommend using an object as a pseudo-enum holding all possible values of `status`:

```js
const STATUS = {
    IDLE: "idle",
    LOADING: "loading",
    ERROR: "error",
    SUCCESS: "success",
};

function Form() {
    const [status, setStatus] = React.useState(STATUS.IDLE);

    // Rest of the component...
}
```

<a href="https://codesandbox.io/s/staging-night-sv7pl" target="_blank">See the full implementation on Codesandbox.</a>

This works even better in TypeScript, where you can enforce type safety by explicitly typing `React.useState`, and you’ll get a compile-time error, when trying to use any other values:

```typescript
type Status = "idle" | "loading" | "error" | "success";

function Form() {
    const [status, setStatus] = React.useState<Status>("idle");
}
```

The rendering logic is similar, but we’re making impossible states really impossible, so we don’t have to worry about those:

```js
function Form() {
    const [status, setStatus] = React.useState(STATUS.IDLE);

    if (status === STATUS.IDLE) {
        return <Form />;
    }

    if (status === STATUS.LOADING) {
        return <Loader />;
    }

    if (status === STATUS.SUCCESS) {
        return <ThankYouMessage />;
    }

    if (status === STATUS.ERROR) {
        return <FormWithErrorMessage />;
    }
}
```

When you have finite states, and you’re trying to model those with multiple boolean values, consider using state enums, as they help avoiding hard-to-catch bugs.

### Further Reading

-   [Stop using isLoading booleans](https://kentcdodds.com/blog/stop-using-isloading-booleans) by Kent C. Dodds
