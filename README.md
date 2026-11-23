# StudyFlow

StudyFlow is a study planning and productivity web application built with React 19 and Vite. It provides a public marketing homepage and an authenticated dashboard for organizing subjects, goals, and tasks, running a focus timer, and reviewing study statistics.

The frontend communicates with a separate backend at `/api` and uses HTTP-only or cookie-based authentication. During local development, Vite proxies `/api` requests to the StudyFlow backend.

## Features

### Public website

- Responsive landing page with product overview
- Feature, workflow, pricing, FAQ, and review sections
- Suggested study plan presentation
- Sign-in and registration pages
- Custom 404 page

### Account and authentication

- Session-aware authentication through `/api/auth/me`
- Email/password login through `/api/auth/login`
- Registration through `/api/auth/register`
- Logout through `/api/auth/logout`
- Protected client routes that redirect unauthenticated users to sign-in
- Toast notifications for authentication and API feedback

### Study dashboard

- **Subjects:** create and delete subjects, assign a custom color, and view completion progress
- **Goals:** create goals with a target, deadline, and related subject
- **Tasks:** create tasks linked to subjects and goals, estimate their duration, set priority and due date, search/filter them, complete or delete them
- **Focus:** select a task and use a 25-minute Pomodoro-style countdown timer
- **Insights:** view overview statistics, weekly task activity, completion trends, and progress by subject

## Tech Stack

| Area | Technology |
| --- | --- |
| UI | React 19, JSX |
| Build tool | Vite 8 |
| Routing | React Router 8 |
| Styling | Tailwind CSS 4, daisyUI 5 |
| Icons | React Icons |
| Charts | Recharts |
| Notifications | React Toastify |
| Linting | ESLint 10 |
| Backend integration | Browser `fetch` API |

## Project Structure

```text
study-flow-main/
├── public/
│   ├── _redirects             # SPA routing redirect configuration
│   ├── favicon.svg
│   └── icons.svg
├── src/
│   ├── assets/                # Images and bundled SVG assets
│   ├── components/            # Homepage and shared UI sections
│   │   └── shared/            # Reusable card and section components
│   ├── context/
│   │   └── AuthContext.jsx    # Authentication session state
│   ├── hooks/
│   │   └── useAuth.jsx        # Authentication context hook
│   ├── layout/
│   │   ├── MainLayout.jsx     # Public navbar/outlet/footer layout
│   │   └── AdminLayout.jsx    # Protected dashboard drawer layout
│   ├── pages/
│   │   ├── homepage/
│   │   ├── SignUp/
│   │   ├── signIn/
│   │   ├── subject/
│   │   ├── goal/
│   │   ├── task/
│   │   ├── focus/
│   │   └── insights/
│   ├── PrivateRoute/
│   │   └── PrivateRoute.jsx   # Route authentication guard
│   ├── index.css              # Tailwind/daisyUI and global styles
│   └── main.jsx               # Application entry point and route setup
├── eslint.config.js
├── vite.config.js
├── package.json
└── index.html
```

## Routes

### Public routes

| Route | Page |
| --- | --- |
| `/` | Marketing homepage |
| `/features` | Features page |
| `/how-it-works` | How-it-works page |
| `/sign-in` | Sign-in form |
| `/sign-up` | Registration form |
| `/private` | Authentication-protected test page |

### Protected dashboard routes

The dashboard is available only when `/api/auth/me` resolves to an authenticated user.

| Route | Module |
| --- | --- |
| `/dashboard/subject` | Subject management |
| `/dashboard/goal` | Goal management |
| `/dashboard/task` | Task management and filtering |
| `/dashboard/focus` | Focus timer |
| `/dashboard/insights` | Study analytics |

Unknown routes render the custom 404 page.

## Getting Started

### Prerequisites

- Node.js 18 or newer
- npm
- Network access to the configured StudyFlow backend

### Installation

```bash
npm install
```

### Run the development server

```bash
npm run dev
```

Vite will print the local URL, typically `http://localhost:5173`.

The development server proxies all `/api` requests to:

```text
https://studyflow-backend-rho.vercel.app
```

This proxy is configured in `vite.config.js`. API requests include credentials so that the backend can return authenticated data based on the browser session.

## Available Scripts

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the Vite development server |
| `npm run build` | Create a production build in `dist/` |
| `npm run preview` | Preview the production build locally |
| `npm run lint` | Run ESLint across the project |

## Backend API

The frontend is designed to work with a REST API served under `/api`. The following endpoints are used:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/api/auth/me` | Get the current authenticated user |
| `POST` | `/api/auth/login` | Sign in |
| `POST` | `/api/auth/register` | Register an account |
| `POST` | `/api/auth/logout` | End the current session |
| `GET`, `POST` | `/api/subjects` | List or create subjects |
| `DELETE` | `/api/subjects/:id` | Delete a subject |
| `GET`, `POST` | `/api/goals` | List or create goals |
| `DELETE` | `/api/goals/:id` | Delete a goal |
| `GET`, `POST` | `/api/tasks` | List or create tasks |
| `DELETE` | `/api/tasks/:id` | Delete a task |
| `PATCH` | `/api/tasks/:id/toggle` | Toggle task completion |
| `GET` | `/api/stats/overview` | Fetch overview dashboard statistics |
| `GET` | `/api/stats/weekly` | Fetch seven-day activity and completion data |
| `GET` | `/api/stats/by-subject` | Fetch subject progress statistics |

API responses are expected to use JSON and commonly include `success`, `message`, and `data` fields. Requests that modify or read private resources use `credentials: "include"`.

### Response shape

Authentication and CRUD features generally expect a shape similar to:

```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": {}
}
```

The exact resource schemas are determined by the external backend. The frontend expects subjects to expose `_id`, `name`, `color`, and progress data; goals to expose `_id`, `title`, `subject`, `deadline`, and completion data; and tasks to expose `_id`, `title`, `subject`, `goal`, `dueDate`, `priority`, `minutes`, and `completed`.

## Configuration

### API proxy

To change the backend used by local development, update `server.proxy['/api'].target` in [vite.config.js](vite.config.js):

```js
server: {
  proxy: {
    "/api": {
      target: "https://studyflow-backend-rho.vercel.app",
      changeOrigin: true,
    },
  },
},
```

For a production deployment, configure the host so `/api` is routed to the backend or replace the `/api` usage with the deployed API URL.

### SPA fallback

`public/_redirects` contains:

```text
/* /index.html 200
```

This supports client-side routes on hosts that recognize Netlify-style redirect syntax.

## Building for Production

```bash
npm run build
npm run preview
```

The optimized static application is emitted into `dist/`. Deploy `dist/` to a static hosting provider and configure an SPA fallback and `/api` routing or proxy so browser routes and backend calls work in production.

## Authentication Flow

1. `AuthContext` calls `/api/auth/me` when the application starts.
2. The authenticated user is stored in React context.
3. `PrivateRoute` waits until session loading completes.
4. Authenticated users are rendered at the protected route.
5. Unauthenticated users are redirected to `/sign-in` with the intended location retained for post-login navigation.
6. Login updates the context immediately and navigates to the previous location or `/`.
7. Logout clears the context user after the backend confirms logout.

## Dashboard Behavior

### Tasks

- Tasks can be filtered by text, completion status, and subject.
- Clicking the checkbox updates the UI optimistically and calls the task toggle endpoint.
- Subject and goal selections are populated from the backend.
- The add-task form requires title, subject, goal, due date, and priority.

### Focus timer

- The focus timer starts at 25 minutes.
- Tasks are loaded from `/api/tasks`.
- Selecting a task resets the timer.
- Start, pause, and reset controls manage the countdown locally.

### Insights

- The overview, weekly, and subject-statistics requests load independently.
- Recharts renders weekly task totals, completed tasks, and completion percentage.
- Subject progress is calculated as a percentage of completed tasks in each subject.

## Development Notes

- The project does not currently include a local backend or database. It depends on the externally hosted API.
- Authentication is session/cookie-based; the frontend does not store a token in local storage.
- Several homepage values and reviews are static sample content.
- Google sign-in and password-reset controls in the UI are currently placeholders and are not wired to backend endpoints.
- The dashboard settings button is currently a visual control without a settings route.
- The external API must permit cookie credentials and matching frontend origin access for authentication to work.

## Deployment Notes

Before deploying, verify all of the following:

1. The backend is reachable over HTTPS.
2. `/api` is proxied or rewritten to the backend in the production environment.
3. CORS and cookie settings allow the production frontend origin.
4. The backend sets cookies for the deployed frontend domain.
5. The host is configured to fall back to `index.html` for React Router paths.

## License

No license is currently specified in this repository. Add a license before redistributing or publishing the project under an open-source license.
