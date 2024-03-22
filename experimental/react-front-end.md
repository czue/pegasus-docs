Standalone React Front End
==========================

*Added in version 2024.4.*

SaaS Pegasus's default React integration is based on a hybrid-model for reasons
[outlined here](https://www.saaspegasus.com/guides/modern-javascript-for-django-developers/client-server-architectures/#enter-the-hybrid-architecture).
The hybrid model is still recommended for the overwhelming majority of Pegasus projects using React.
However, there are valid reasons to run a completely separate React front---including access to dedicated tooling and libraries,
and isolating your front end and back end code.

Pegasus experimentally ships with a decoupled front end *example application* that can be used as a starting point for building
out a decoupled front end with React.
It uses [Vite](https://vitejs.dev/) as a development server and build tool.

The features it includes are:

- A standalone Vite / React application.
- Authentication via JWT---including sign up, login and logout functionality.
- A sample profile page which shows how to retrieve data from your back end and display it.
- The employee lifecycle demo that ships with Pegasus (if enabled).

The standalone front end is *only available on TailwindCSS* and uses DaisyUI for styling.

**The standalone is not intended to be a replacement for Pegasus's UI, but a reference example you can use
as a starting point to build standalone, React, single-page-applications with Pegasus.**

Here are some technical details:

## Running the front end

*If you are using Docker, your front end should start in a separate container after running `make init`.*

The front end lives in the `/frontend` folder of your project.
To set it up for the first time, first go into the directory:

```
cd frontend
```

And install npm packages:

```
npm install
```

Create your `.env` file:

```bash
cp .env.example .env
```

Then run the dev server:

```
npm run dev
```

Note: your Django backend must also be running for the front end to work,
and you must also [build your Django front end](front-end.md) for styles to work.


## Authentication

Authentication is handled via *protected routes* and *authentication context*.
You can see an example of how to set this up in the profile page.

Any page in your application that requires login can be wrapped in the `ProtectedRoute` component.
For example, like this:

```jsx
<ProtectedRoute>
  <p>Hello authenticated user!</p>
</ProtectedRoute>
```

Alternatively, if you make a page a child of the `<Dashboard>` component this will be automatically configured for you.
See `main.tsx` as an example of how this is set up.

When using the `ProtectedRoute`, if the user is not logged in they will be redirected to the login page.
If they are logged in, they will be able to access the route, and you can assume access
to the user object and other APIs.

If you want to access user data you can use the `AuthContext` context.
This context is made available from the `AuthProvider` component, which is available on all pages.

Here is an example of using the `AuthContext` from the Profile page:

```jsx
import {useContext} from "react";
import {AuthContext} from "../../auth/authcontext";


export default function Profile() {
  const { user } = useContext(AuthContext);
  return (
    <p>
      Hello {user?.getDisplayName}!
    </p>
  );
}
```

## Backend API access

The front end uses the [same api client](apis.md#api-clients) as the backend / hybrid model.
The API client is installed as a local npm package.
If the APIs are not public, it should be initialized with an authentication token (which you can get from the `AuthContext`).
Here is a basic example from the employee app demo:

```jsx
import {useContext} from "react";
import {AuthContext} from "../../auth/authcontext.tsx";
import {PegasusApi} from "api-client";

export default function EmployeeApp() {
  const {token} = useContext(AuthContext);
  const client = new PegasusApi(getApiConfiguration(token));

  return (
    <EmployeeApplication client={client} />
  );
}
```

## Routing

Routing is handled by [React Router](https://reactrouter.com/en/main).

The main routes for the project are configured in `main.tsx`, and you can also include child routes
by following the pattern used by the employee demo.

## Known Limitations

This is an experimental feature meant to provide a starting point for building a standalone React front end against your Pegasus app.
It is *not* a complete, production-ready app, in the same way that standard Pegasus is.

Here are some of the larger limitations:

- Only a very limited subset of Pegasus functionality is available in the front end.
- The front end styles only support Tailwind CSS.
- There is no guidance/support for production deployment of the front end.

## Troubleshooting

**I'm getting a "URI malformed" error when I load the app.**

This is likely because your `frontend/.env` file does not exist, or your `VITE_APP_BASE_URL` is not
properly set inside it. See `frontend/.env.example` for an example `.env` file suitable for development.

## Feedback

If you have any feedback on this feature I would love to hear it!
Feedback could include bug reports, feature requests, or any suggested architectural changes. 
