
# 🚀 Onboarding Project: Auth & Task Dashboard Spec

**Target Stack:** Angular v22+, TypeScript, CSS/SCSS

**Architecture:** Standalone Components, Signals, Signal Forms, Functional Guards, `httpResource`

---

## 🎯 Project Overview

You will build a single-page application featuring an authentication flow and a protected dashboard. The goal is to get hands-on experience with modern Angular patterns—specifically how Angular handles state, forms, routing, and data fetching **without** legacy `NgModule` boilerplate.

---

## 📋 Acceptance Criteria Checklist

### Phase 1: Authentication State (`AuthService`)

* [ ] Create an `AuthService` provided at the root level.
* [ ] Store the current user state using a writable `signal<User null |>(null)`.
* [ ] Expose a `currentUser` read-only signal to components.
* [ ] Create a `computed()` signal named `isAuthenticated` that returns `true` if `currentUser` is not `null`.
* [ ] Implement a `login(user, token)` method that sets state and saves a mock token to `localStorage`.
* [ ] Implement a `logout()` method that clears state and removes the token from `localStorage`.

> 💡 **React Bridge Note:** An Angular `@Injectable()` service acts like a combination of a React Context Provider and a custom hook, but without requiring component wrapping or trigger re-renders up the tree.

---

### Phase 2: Login Component & Signal Forms (`/login`)

* [ ] Create a standalone `LoginComponent` assigned to the `/login` route.
* [ ] Define backing form model state using `signal({ email: '', password: '' })`.
* [ ] Construct a schema-validated form tree using `form()` from `@angular/forms/signals`.
* [ ] Implement validation rules:
* **Email:** Required, valid email format.
* **Password:** Required.


* [ ] Bind HTML inputs using the `[formField]` directive.
* [ ] Display dynamic validation error messages **only** if the field is `touched()` and `invalid()`.
* [ ] Disable the submit button whenever the form is `invalid()`.
* [ ] On submission:
* Prevent default browser form submission.
* Call `AuthService.login()` with mock user details.
* Programmatically navigate to `/dashboard` using Angular's `Router`.



> 💡 **React Bridge Note:** Signal Forms handle two-way data sync and validation state out of the box. Reading field states like `myForm.email().invalid()` gives you fine-grained reactive updates similar to `react-hook-form`.

---

### Phase 3: Route Protection (`authGuard`)

* [ ] Create a functional route guard (`CanActivateFn`) named `authGuard`.
* [ ] Inject `AuthService` and `Router` using Angular’s `inject()` function.
* [ ] Check `authService.isAuthenticated()`:
* If `true`, allow navigation.
* If `false`, redirect the user back to `/login` using `router.createUrlTree(['/login'])`.


* [ ] Attach this guard to the `/dashboard` route definition in `app.routes.ts`.

---

### Phase 4: Protected Dashboard & Data Fetching (`/dashboard`)

* [ ] Create a standalone `DashboardComponent` protected by `authGuard`.
* [ ] Render a personalized welcome message reading `AuthService.currentUser()`.
* [ ] Add a "Log Out" button in the header that invokes `AuthService.logout()` and routes back to `/login`.
* [ ] Use `httpResource` to fetch a list of sample tasks from:
`[https://jsonplaceholder.typicode.com/todos?_limit=5](https://jsonplaceholder.typicode.com/todos?_limit=5)`
* [ ] Build the UI template using modern control flow syntax:
* Use `@if` to render a loading message while `resource.isLoading()` is true.
* Use `@else if` to render an error message if `resource.error()` occurs.
* Use `@for` to loop over `resource.value()` and display each task item with mandatory `track task.id`.



> 💡 **React Bridge Note:** `httpResource()` is Angular’s native answer to `useQuery` / TanStack Query. It returns reactive signals for `.value()`, `.isLoading()`, and `.error()`.

---

### Phase 5: Stretch Goal — State Persistence

* [ ] Update `AuthService` to check `localStorage` when initialized.
* [ ] If a token exists, automatically restore user state so refreshing the page on `/dashboard` does not log the user out.

---

## 🛠️ Definition of Done

1. Zero `NgModule` usage (all components/services use Standalone APIs).
2. Clean separation of concern: state/HTTP calls inside services, UI logic inside components.
3. No legacy template syntax (`*ngIf`, `*ngFor`). Only `@if` and `@for`.
4. App handles unauthenticated direct navigation to `/dashboard` cleanly (redirects to `/login`).

---

Here is a **PR Review Rubric & Testing Checklist** tailored for reviewing this onboarding project. It focuses on modern Angular practices and catching common habits React developers bring over when learning Angular.

---

## 🔍 Code Review Rubric

### 1. Modern Angular Architecture & Control Flow

| Category | What to Look For (Pass ✅) | Red Flags / Antipatterns (Fail ❌) |
| --- | --- | --- |
| **Component Setup** | Imports components/directives directly in `@Component({ imports: [...] })`. | Declaring or using any `NgModule`. |
| **Template Flow** | Uses modern `@if`, `@else`, and `@for` syntax. | Using legacy `*ngIf` or `*ngFor` directives. |
| **Loop Tracking** | `@for (item of items; track item.id)` always includes a `track` expression. | Missing `track` statement in `@for`. |
| **Dependency Injection** | Uses `private authService = inject(AuthService);` | Constructor parameter injection unless explicitly requested. |

---

### 2. Signals & Reactivity

| Category | What to Look For (Pass ✅) | Red Flags / Antipatterns (Fail ❌) |
| --- | --- | --- |
| **Signal Usage** | Reads signals with parens: `user()`, `isAuthenticated()`. | Omitting parens `user` (passing the signal wrapper instead of value). |
| **State Protection** | Exposes state via `.asReadonly()` from services. | Exposing raw writable signals (`WritableSignal`) directly to components. |
| **Derived State** | Uses `computed()` for values derived from other signals. | Manually recalculating values inside a `signal()` update or template method call. |

---

### 3. Forms & Data Fetching

| Category | What to Look For (Pass ✅) | Red Flags / Antipatterns (Fail ❌) |
| --- | --- | --- |
| **Signal Forms** | Schema built with `form()`, bound via `[formField]`. | Reverting to legacy `ReactiveFormsModule` / `FormBuilder` or `ngModel`. |
| **Validation UX** | Errors display conditionally when field is `touched() && invalid()`. | Showing validation errors instantly before the user touches the field. |
| **HTTP Queries** | Uses `httpResource()` for fetching data in components. | Subscribing to raw `HttpClient` observables in component TS code without unsubscription. |

---

## 🧪 Manual QA Testing Checklist

Copy and test these scenarios on their running application:

### Scenario 1: Unauthenticated Navigation

* [ ] Navigate directly to `http://localhost:4200/dashboard` in an incognito window.
* [ ] **Expected Result:** App instantly redirects to `/login`.

### Scenario 2: Form Validation

* [ ] Type an invalid email (`test@com`) into the email field and blur out of the field.
* [ ] **Expected Result:** Validation message displays. Login button remains disabled.
* [ ] Type a valid email and password.
* [ ] **Expected Result:** Validation message disappears. Login button enables.

### Scenario 3: Auth & Protected Dashboard

* [ ] Click "Log In".
* [ ] **Expected Result:** Router navigates to `/dashboard`. User welcome message displays the logged-in email/name.

### Scenario 4: Resource States

* [ ] Inspect network tab or set throttle to "Fast 3G".
* [ ] Refresh dashboard page (or log in).
* [ ] **Expected Result:** Loading message briefly displays before task list renders. No console errors.

### Scenario 5: Logout Flow

* [ ] Click "Log Out".
* [ ] **Expected Result:** App routes to `/login`.
* [ ] Press browser back button.
* [ ] **Expected Result:** Route guard blocks re-entry to `/dashboard` and keeps user on `/login`.

### Scenario 6: Persistence (Stretch Goal)

* [ ] Log in, refresh browser tab on `/dashboard`.
* [ ] **Expected Result:** User remains logged in without being kicked to `/login`.

---

## 💡 Common React-to-Angular Coaching Notes

When leaving review comments, these pointers help bridge the React mental model:

> 💬 **If they write custom functions instead of Services:**
> *"In React, you'd use a custom hook here. In Angular, singletons provided via `@Injectable({ providedIn: 'root' })` are the idiomatic way to hold shared state across routes."*

> 💬 **If they call methods inside template expressions:**
> *"Calling `getUserName()` in the template runs on every change detection cycle. Use a `computed()` signal instead so Angular only recomputes it when the underlying signal changes."*

> 💬 **If they manually subscribe to HTTP calls:**
> *"Rather than `.subscribe()`, prefer `httpResource()`. It automatically manages loading, error, and data signals, similar to `useQuery` in React Query."*

**Yes, absolutely.** `angular.dev` is hands-down the best place to send them.

When Angular revamped its documentation site, they built it specifically to teach **modern, standalone, signal-based Angular**. Unlike the old `angular.io` site (which was notorious for throwing legacy boilerplate at newcomers), `angular.dev` feels fresh, modern, and very friendly to developers coming from React.

---

## Why `angular.dev` Works So Well for React Devs

* **Interactive Playground (`angular.dev/tutorials`):** They can run and edit code in an embedded browser environment without having to set up a local CLI or node modules first.
* **Modern by Default:** Everything on the site defaults to Standalone Components, Signals, and `@if`/`@for` control flow. They won't accidentally stumble into 2018-era `NgModule` tutorials.
* **Deep Dive Guides:** The guides for **Signals**, **Routing**, and **Data Fetching** are concise and focus on *why* things work the way they do rather than just dumping boilerplate.

---

## Specific Pages to Bookmark for Them

When you hand them the docs, point them directly to these sections so they don't get lost:

1. **[Angular Essentials Tutorial](https://angular.dev/tutorials/first-app):** A quick, hands-on walkthrough that covers components, inputs/outputs, and signals in under 30 minutes.
2. **[Signals Guide](https://angular.dev/guide/signals):** Essential reading. It explains `signal()`, `computed()`, and `effect()`, which will instantly make sense to anyone who knows React state and `useMemo`.
3. **[Control Flow Guide](https://angular.dev/guide/templates/control-flow):** Shows the `@if`, `@else`, and `@for` syntax, bridging the gap from JSX conditionals.

---

## One Crucial Warning to Give Them

> **"If you Google a problem, ignore any article written before 2024 or anything that mentions `NgModule` or `*ngIf`."**

The biggest trap for a junior dev learning Angular today is landing on 6-year-old StackOverflow posts or Medium articles teaching legacy patterns. Remind them: if a tutorial tells them to import `BrowserModule` or create an `@NgModule`, it's outdated—stick to `angular.dev` or your team's code review notes.
