Standalone React (Next.js) Front End
====================================

SaaS Pegasus's default React integration is based on a hybrid-model for reasons
[outlined here](https://www.saaspegasus.com/guides/modern-javascript-for-django-developers/client-server-architectures/#enter-the-hybrid-architecture).
The hybrid model is still recommended for the overwhelming majority of Pegasus projects using React.
However, there are valid reasons to run a completely separate React front---including access to dedicated tooling and libraries,
and isolating your front end and back end code.

Pegasus experimentally ships with a decoupled front end *example application* that can be used as a starting point for building
out a decoupled front end with React and Next.js.

The features it includes are:

- A standalone Next.js application.
- Authentication via JWT---including sign up, login and logout functionality.
- A sample profile page which shows how to retrieve data from your back end and display it.

The decoupled front end is *only available on TailwindCSS* and uses DaisyUI for styling.

Here are some technical details:

## Running the front end

The front end lives in the `/frontend` folder of your project.
To set it up for the first time, first go into the directory:

```
cd frontend
```

And install npm packages:

```
npm install
```

Then run the dev server:

```
npm run dev
```

Note: your Django backend must also be running for the front end to work.

## Authentication

Authentication is handled via *protected routes* and *authentication context*.
You can see an example of how to set this up in the profile page.

Any page in your application that requires login can be wrapped in the `ProtectedRoute` component.
If your pages are underneath the `/dashboard/` route/folder this will happen automatically.
Otherwise you can wrap things like this in your `layout.tsx` file.

```jsx
<ProtectedRoute>
  {children}
</ProtectedRoute>
```

If the user is not logged in they will be redirected to the login page.
If they are logged in, they will be able to access the route, and you can assume access
to the user object and other APIs.

If you want to access user data you can use the `AuthContext` context.

Here is an example:

```jsx
'use client'
import {useContext} from "react";
import {AuthContext} from "@/auth/authcontext";

export default function Page() {
  const { user } = useContext(AuthContext);
  return (
    <p>
      Hello {user?.getDisplayName}
    </p>
  );
}
```

## Backend API access

The front end uses the [same api client](apis.md#api-clients) as the backend / hybrid model.
The API client is installed as a local npm package. You can access it like the following:

```jsx
todo example
```
