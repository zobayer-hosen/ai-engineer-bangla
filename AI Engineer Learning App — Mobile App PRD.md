# AI Engineer Learning App — Mobile App PRD

Oct 12, 2026 · @BEE SMART

## 1. Document scope

This PRD covers only the learner-facing mobile app (Android + iOS, React Native + Expo). It is the first of three PRDs split from the original combined spec:

| PRD | Covers | Status |
| --- | --- | --- |
| Mobile App PRD (this doc) | Learner app screens, flows, client architecture, offline and security on device | Draft v1.0 |
| Web PRD | Learner web app + admin/content dashboard | Next |
| Backend PRD | Go + Gin API, PostgreSQL schema, auth, scoring, AI Tutor proxy | After web |

The app is a thin, secure client. All business rules — authorization, quiz scoring, progress calculation, AI provider calls — live in the Go backend. The app never holds secrets and never computes a value the server must trust.

Out of scope here: admin/content management UI (Web PRD), database and endpoint implementation (Backend PRD).

## 2. Product overview

The app gives beginners a guided, mobile-first path from Python basics to job-ready AI Engineering, in short lessons they can finish in 10–15 minutes.

**Vision.** A learner opens the app and always knows the one next thing to do — read a lesson, take a quiz, or review a weak prerequisite.

**Problems the app solves**

- Learners don't know what to study first or in what order → a prerequisite-based roadmap with locked/unlocked stages.
- Theory and practice feel disconnected → every lesson ends with a quiz or exercise.
- Progress is invisible → server-calculated progress, streaks and course completion.
- No one to ask when stuck → AI Tutor (Phase 2), aware of the current lesson.

**App goals**

1. A new user reaches their first lesson within 3 minutes of installing.
2. A returning user resumes their last lesson in one tap from Home.
3. Lessons read comfortably on a 5-inch phone, including code blocks.
4. The app works on unstable mobile networks: cached content, clear errors, safe retries.

## 3. Target users and personas

The primary user is a CSE student or self-learner who studies on a phone in short sessions, often on mobile data.

| Persona | Starting level | Main need in the app | Starts at |
| --- | --- | --- | --- |
| Rafi, 2nd-year CSE student | Beginner: some Python, no ML | Clear order of topics, daily habit | Stage 1 or 2 |
| Nusrat, self-learner switching careers | Beginner: no programming | Small lessons, simple explanations | Stage 1 |
| Tanvir, web developer | Intermediate: strong coding, weak math | Skip basics, fill math gaps, reach ML fast | Stage 2 or 4 |
| Mim, ML practitioner | Advanced | GenAI, RAG, deployment topics | Stage 8 or 9 |

During onboarding the learner picks a level and goal; the app recommends a starting stage but never locks the learner out of earlier stages for review.

## 4. Release scope

MVP ships the core loop — onboard, pick a course, read a lesson, pass a quiz, see progress — and nothing that needs a sandbox or an AI provider.

| Feature | MVP (v1.0) | Phase 2 (v1.x) | Phase 3 (v2) |
| --- | --- | --- | --- |
| Register, login, logout, password reset | Yes |  |  |
| Onboarding: level + goal | Yes |  |  |
| Profile view/edit | Yes |  |  |
| Roadmap (10 stages) | Yes |  |  |
| Course catalog + details + enroll | Yes (list + filter by stage/difficulty) | Full-text search, recommendations |  |
| Text lessons with code blocks, images | Yes | Video, downloads | Offline lessons |
| Lesson completion + resume | Yes |  |  |
| Multiple-choice quizzes, server scoring, explanations | Yes |  |  |
| Coding practice | Static code + MCQ code questions |  | Sandboxed execution |
| Home dashboard + streak | Yes (basic) | Daily goal settings |  |
| Progress screen | Yes | Advanced analytics |  |
| AI Tutor |  | Yes |  |
| Push reminders, notification center |  | Yes |  |
| Capstone projects, certificates |  |  | Yes |
| Dark mode | Yes |  |  |
| Language: Bangla only (UI + all content) | Yes | No English version planned |  |

## 5. Information architecture and navigation

The app has two route groups: a signed-out auth stack and a signed-in bottom tab bar with five tabs; detail screens (course, lesson, quiz) push on top of the tab that opened them.

&#91;embedded content: app navigation · auth stack + 5 tabs\]

Deep links (`aiengineer://lesson/:id`, `aiengineer://course/:id`) open the matching screen after login. Android back button and iOS swipe-back always return to the previous screen; leaving an unsubmitted quiz asks for confirmation.

## 6. Screen-by-screen requirements

Each requirement has an ID (APP-xx) for tickets and QA. Priority: P0 = MVP blocker, P1 = MVP if time allows, P2 = later phase.

### 6.1 Splash and onboarding

- **APP-01 (P0)** Splash checks for a stored refresh token; valid → Home, missing/invalid → Onboarding or Login.
- **APP-02 (P0)** Onboarding: 3 swipeable intro cards (roadmap, lessons + quizzes, track progress), Skip on every card.
- **APP-03 (P0)** After registration, ask experience level (Beginner / Intermediate / Advanced) and goal (e.g. ML Engineer, GenAI Engineer, just exploring). Saved to profile.
- **APP-04 (P0)** Show the recommended starting stage with a "Start here" button and a "Choose another stage" link.

### 6.2 Authentication

- **APP-05 (P0)** Register: name, email, password, confirm password. Client-side checks (email format, password ≥ 8 chars) before submit; server errors shown inline under the field.
- **APP-06 (P0)** Login: email + password, show/hide password, loading state on the button, disabled while submitting.
- **APP-07 (P0)** Forgot password: email input → confirmation screen that never reveals whether the email exists.
- **APP-08 (P0)** Tokens stored only in expo-secure-store; refresh handled silently; on refresh failure the user returns to Login with a "Session expired" message.
- **APP-09 (P1)** Google sign-in (open question: needs backend support).

### 6.3 Home dashboard

- **APP-10 (P0)** Greeting with first name.
- **APP-11 (P0)** Continue Learning card: last opened lesson, course name, progress bar, one-tap resume. It is the most prominent element on the screen.
- **APP-12 (P0)** Current roadmap stage + overall progress percentage (from server).
- **APP-13 (P0)** Streak counter (days in a row with at least one completed lesson or quiz).
- **APP-14 (P1)** Recommended next lesson and recently completed lessons (max 3 each).
- **APP-15 (P0)** Empty state for new users: "Start your first lesson" pointing to the recommended course.
- **APP-16 (P0)** Pull to refresh.

### 6.4 Learning roadmap

- **APP-17 (P0)** Vertical list of the 10 stages with name, course count, and status: Not started / In progress / Completed.
- **APP-18 (P0)** Stages are recommended in order but never hard-locked; a stage with unmet prerequisites shows a "Recommended first: Stage N" hint.
- **APP-19 (P0)** Tapping a stage opens its courses.

### 6.5 Course catalog

- **APP-20 (P0)** Paginated list (infinite scroll) of published courses: title, stage, difficulty, duration, enrolled badge.
- **APP-21 (P0)** Filters: stage, difficulty. Filter chips persist while the screen is open.
- **APP-22 (P2)** Search by title/topic with 300 ms debounce.

### 6.6 Course details

- **APP-23 (P0)** Title, description, author, difficulty, estimated duration, prerequisites (tappable), learning objectives.
- **APP-24 (P0)** Module list, expandable to lessons, each lesson with a completed check.
- **APP-25 (P0)** Enroll button; after enrollment it becomes "Continue" and completion % appears.
- **APP-26 (P0)** Enrollment is idempotent: double taps or retries never create duplicate enrollments.

### 6.7 Lesson screen

- **APP-27 (P0)** Render lesson content from Markdown: headings, paragraphs, lists, images, tables, and code blocks with syntax highlighting, horizontal scroll and a Copy button.
- **APP-28 (P0)** Key takeaways box at the end.
- **APP-29 (P0)** "Mark as complete" button; shows confirmed state only after the server accepts it.
- **APP-30 (P0)** Save scroll position; reopening the lesson resumes there.
- **APP-31 (P0)** Previous / Next lesson navigation; Next after the last lesson leads to the module quiz if one exists.
- **APP-32 (P1)** Adjustable font size (3 steps).
- **APP-33 (P2)** Embedded video and downloadable resources.
- **APP-34 (P2)** "Ask AI Tutor about this lesson" button.

### 6.8 Coding practice

- **APP-35 (P0)** List of exercises per course with done state.
- **APP-36 (P0)** MVP exercise types: read-the-code MCQ ("what does this print?"), fill-the-blank with options, spot-the-bug with options. Graded by the server like quiz questions.
- **APP-37 (P2)** Code editor with monospace font, tab key bar and Run/Submit, only after the sandbox service exists. The app shows exactly what the server returns; it never runs user code locally.

### 6.9 Quiz and results

- **APP-38 (P0)** One question per screen, progress indicator (3/10), single-answer MCQ.
- **APP-39 (P0)** Answers held locally until Submit; leaving mid-quiz asks for confirmation.
- **APP-40 (P0)** Submit sends selected option IDs only; score, pass/fail and explanations come from the server response.
- **APP-41 (P0)** Results: score, pass mark, per-question correct answer + explanation, Retry (if the retry policy allows) and Back to course.
- **APP-42 (P1)** Attempt history per quiz.

### 6.10 My Learning and progress

- **APP-43 (P0)** My Learning tabs: In progress / Completed courses.
- **APP-44 (P0)** Progress screen: overall %, per-stage %, lessons completed, quizzes passed, current and longest streak.
- **APP-45 (P2)** Weekly activity chart and learning time.

### 6.11 AI Tutor (Phase 2)

- **APP-46 (P2)** Chat screen with conversation list; lesson context attached when opened from a lesson.
- **APP-47 (P2)** Streaming responses, code blocks rendered like lessons, label "Not executed — explanation only" on generated code.
- **APP-48 (P2)** Show remaining daily messages; friendly message when the limit is reached.

### 6.12 Notifications (Phase 2)

- **APP-49 (P2)** Ask notification permission after the first completed lesson, not at launch.
- **APP-50 (P2)** Daily reminder at a user-chosen time; streak-at-risk reminder; notification center list with read state.

### 6.13 Profile and settings

- **APP-51 (P0)** View/edit name, level, goal.
- **APP-52 (P0)** Theme: System / Light / Dark.
- **APP-53 (P0)** Logout (clears secure storage and cached user data).
- **APP-54 (P0)** Delete account request (required by App Store and Google Play policy).
- **APP-55 (P0)** About, privacy policy, terms, app version.

## 7. Key user flows

These four flows are the MVP's critical journeys; each becomes an end-to-end test.

**Flow A — First launch to first lesson (target: under 3 minutes)**

1. Splash → Onboarding cards (or Skip).
2. Register → level + goal.
3. Recommended stage → recommended course → Enroll.
4. First lesson opens directly.

**Flow B — Daily return**

1. Open app → Splash refreshes the session silently.
2. Home → Continue Learning card → lesson resumes at saved scroll position.
3. Mark as complete → Next lesson, streak updates.

**Flow C — Module quiz**

1. Finish last lesson of a module → Next → Quiz intro (questions count, pass mark).
2. Answer questions → Submit.
3. Server returns score → Results with explanations.
4. Passed → module marked complete, back to course. Failed → Retry (if allowed) or Review lessons.

**Flow D — Session expired**

1. Any API call returns 401 → app tries one refresh.
2. Refresh succeeds → original request repeats automatically.
3. Refresh fails → secure storage cleared, Login with "Session expired"; after login the user returns to the screen they were on.

## 8. UI/UX and design system

One small, consistent component set is built first and every screen uses it; no screen-specific one-off styles.

**Design tokens**

- Colors: primary, surface, text-primary/secondary, success, warning, error — each defined for light and dark.
- Typography: 4 sizes (title, heading, body 16, caption); monospace for code.
- Spacing on an 8-point scale; corner radius 8–12; touch targets ≥ 44×44 pt.

**Core components**

- Button (primary/secondary/ghost, loading, disabled), TextField with error text, Card, ProgressBar, Badge (difficulty, enrolled), Chip (filters), ListItem, EmptyState, ErrorState, Skeleton loader, CodeBlock, Markdown renderer, Toast.

**Every data screen handles four states**

| State | What the user sees |
| --- | --- |
| Loading | Skeleton placeholders shaped like the content, not a full-screen spinner |
| Empty | Illustration-free message + one action (e.g. "Browse courses") |
| Error | Short reason + Retry button; cached data stays visible if present |
| Offline | Top banner "You're offline — showing saved content"; actions that need the server are disabled with a hint |

**Lesson reading**

- Body text 16–18 pt, line height \~1.5, max one idea per paragraph.
- Code blocks never wrap; they scroll horizontally with a visible Copy button.

Open question: brand name, logo and primary color are not decided yet.

### 8.1 Language: Bangla only

Every word a learner sees in the app is in Bangla; there is no English UI and no language switch.

- **APP-56 (P0)** All UI text in Bangla: buttons, menus, onboarding, errors, empty states, toasts, notifications, store listing, privacy policy and terms.
- **APP-57 (P0)** All learning content in Bangla: lessons, key takeaways, quiz questions, options, explanations, exercise prompts, AI Tutor replies.
- **APP-58 (P0)** Numbers and dates shown in Bangla: Bengali digits (০–৯) for progress, scores, streaks; Bangla month names (e.g. ১২ অক্টোবর ২০২৬).
- **APP-59 (P0)** Bangla font bundled with expo-font (Noto Sans Bengali or Hind Siliguri), so text looks the same on every phone; line height \~1.6–1.7 for body text because Bangla conjuncts and vowel signs need more room.
- **APP-60 (P0)** Server error messages are mapped from error codes to Bangla text in the app; the app never shows a raw English message from the server or the OS.
- **APP-61 (P0)** Text inputs accept Bangla typing (Avro, Gboard Bangla); names can be in Bangla.

**Unavoidable exceptions** (these are code, not language): Python/library keywords and names inside code blocks (`import numpy`, `def`, `print`), file names and terminal commands, and the email/password a user types. Code comments and printed-output explanations in lessons are written in Bangla.

All UI strings still live in one `bn` locale file (not hard-coded in screens), so wording can be fixed in one place.

## 9. Mobile technical architecture

The app is a TypeScript Expo project with file-based routing, server state in TanStack Query, and a single typed API client generated from the backend's OpenAPI spec.

| Concern | Choice | Why |
| --- | --- | --- |
| Framework | React Native + Expo (managed workflow, latest stable SDK) | One codebase for Android + iOS, OTA updates |
| Language | TypeScript, strict mode | Fewer runtime bugs |
| Navigation | Expo Router (tabs + stacks) | File-based, deep links for free |
| Server state | TanStack Query | Caching, retries, pagination, offline persistence |
| Client state | Zustand (auth status, theme, quiz-in-progress) | Small and simple |
| Forms | react-hook-form + zod | Validation shared by all forms |
| HTTP | Axios or fetch wrapper with auth + refresh interceptor | One place for tokens and errors |
| API types | Generated from OpenAPI (openapi-typescript) | App and backend never drift |
| Secure storage | expo-secure-store | Refresh token in Keychain / Keystore |
| Cache persistence | AsyncStorage / MMKV for query cache | Show last content when offline |
| Markdown + code | react-native-markdown-display + syntax highlighter | Lesson rendering |
| Push (Phase 2) | expo-notifications | Reminders |
| Errors | Sentry (expo integration) | Crash and error reports |
| Build/release | EAS Build, EAS Submit, EAS Update | Store builds and hotfixes |

**Folder structure** (inside `mobile/`)

```text
mobile/
├── app/                  # Expo Router routes
│   ├── (auth)/           # login, register, forgot-password, onboarding
│   ├── (tabs)/           # home, learn, practice, progress, profile
│   ├── course/[id].tsx
│   ├── lesson/[id].tsx
│   └── quiz/[id].tsx
├── features/             # auth, courses, lessons, quizzes, progress, tutor
│   └── <feature>/        # api.ts, hooks.ts, components/, types.ts
├── components/ui/        # design-system components
├── services/api/         # client, interceptors, generated types
├── store/                # zustand stores
├── theme/                # tokens, light/dark
├── utils/
└── app.config.ts         # env per build profile
```

**Rules**

- No API keys or secrets in the bundle; the only config is the API base URL per environment (dev / staging / prod).
- Access token in memory only; refresh token in secure storage.
- Each feature folder owns its queries and mutations; screens never call the HTTP client directly.

## 10. API dependencies

The app needs these endpoints from the Go backend; exact request/response shapes will be fixed in the Backend PRD and OpenAPI spec.

| Screen / feature | Endpoints | Phase |
| --- | --- | --- |
| Auth | `POST /auth/register`, `/auth/login`, `/auth/refresh`, `/auth/logout`, `/auth/forgot-password` | MVP |
| Profile, onboarding | `GET /me`, `PATCH /me` | MVP (new, not in original spec) |
| Roadmap | `GET /roadmap` (stages + user status) | MVP (new) |
| Catalog, details | `GET /courses?stage=&difficulty=&page=`, `GET /courses/:id`, `GET /courses/:id/modules` | MVP |
| Enroll | `POST /courses/:id/enroll` (idempotent) | MVP |
| Lesson | `GET /lessons/:id`, `PUT /lessons/:id/progress` (scroll position, completed) | MVP |
| Quiz | `GET /quizzes/:id`, `POST /quizzes/:id/attempts`, `GET /quiz-attempts/:id` | MVP |
| Home, progress | `GET /me/dashboard` (one call for Home), `GET /me/progress`, `GET /me/courses`, `GET /me/activity` | MVP (`/me/dashboard` new) |
| Account deletion | `DELETE /me` or deletion request | MVP (new) |
| AI Tutor | `POST /ai-tutor/messages` (streaming), `GET /ai-tutor/conversations[/:id]` | Phase 2 |
| Push | `POST /me/devices` (Expo push token), notification preferences | Phase 2 |

All paths are prefixed `/api/v1`. The app expects one error format, e.g. `{ "error": { "code": "VALIDATION_ERROR", "message": "...", "fields": {...} } }`, so it can show field-level errors.

## 11. Non-functional requirements

**Security**

- HTTPS only; no cleartext traffic allowed in Android/iOS config.
- Refresh token in secure storage, access token in memory; both cleared on logout and on account deletion.
- No secrets, AI keys or admin features in the app bundle.
- Never log tokens, passwords or tutor messages to console or crash reports.
- Treat every server ID as untrusted input in deep links; the server enforces ownership.

**Performance (targets to validate on a mid-range Android phone)**

- Cold start to interactive Home: ≤ 3 s on 4G.
- Cached screens render instantly, then refresh in the background.
- Lists are virtualized (FlashList/FlatList) and paginated (20 items per page).
- Images sized for the device and cached.
- App download size kept small; no unused heavy libraries.

**Offline and reliability**

- Last viewed courses and lessons stay readable offline from the query cache.
- "Mark complete" and quiz submit require network; failures show Retry and never show a fake success.
- Progress updates and enrollments are safe to retry (server is idempotent).
- Quiz answers in progress survive app backgrounding.

**Accessibility**

- Labels for screen readers on every button and icon (TalkBack, VoiceOver).
- Supports system font scaling up to 200% without broken layouts.
- Text contrast ≥ 4.5:1 in both themes; status never shown by color alone.

**Compatibility**

- Android 8.0+ (API 26) and iOS 15+ (confirm against the chosen Expo SDK).
- Phones first; tablets supported in portrait with a centered max-width layout.

**Privacy**

- Privacy policy linked in-app and in store listings; Play Data Safety and App Store privacy labels filled accurately.
- Analytics events contain no lesson answers or tutor message text.

## 12. Analytics and success metrics

The app sends a small set of product events so the metrics below can be measured; numeric targets are set after 4–6 weeks of baseline data.

| Event | Fired when | Feeds metric |
| --- | --- | --- |
| `onboarding_completed` | Level + goal saved | Registration completion rate |
| `course_enrolled` | Enroll succeeds | Enrollment rate |
| `lesson_opened` / `lesson_completed` | Lesson screen opens / server confirms completion | Lesson completion rate |
| `quiz_submitted` | Server returns result (with pass/fail) | Quiz participation and pass rate |
| `app_opened` | App comes to foreground with a session | Weekly active learners, retention (D1, D7, D30) |
| `course_completed` | Course reaches 100% | Course completion rate |
| `tutor_message_sent` (Phase 2) | Message accepted | Tutor usage |

Tool choice (PostHog, Firebase Analytics, or backend-only events) is an open question; events are named the same either way.

## 13. Testing and release

**Testing**

- Unit: Jest for utils, stores, form schemas, token refresh logic.
- Component: React Native Testing Library for core UI components and the four data states.
- End-to-end: Maestro (or Detox) for Flows A–D in section 7, run against a staging backend.
- Manual device matrix before each release: 1 low-end Android, 1 recent Android, 1 iPhone, light + dark, 200% font size.

**Release**

- EAS build profiles: development, preview (internal testers), production.
- Internal testing via Google Play internal track and TestFlight before public release.
- EAS Update for JS-only fixes; native changes go through a store build.
- Store assets needed: app name, icon, screenshots, short/long description, privacy policy URL, Data Safety form, App Privacy labels, demo account for App Review.

## 14. MVP acceptance criteria

The app MVP is ready for store release when every item below passes on Android and iOS against the staging backend.

- [ ] A new user can register, complete onboarding and open a first lesson in under 3 minutes.
- [ ] Login persists across app restarts; an expired session returns the user to Login with a clear message.
- [ ] Roadmap shows all 10 stages with correct status from the server.
- [ ] Catalog lists published courses with working stage and difficulty filters and pagination.
- [ ] A learner can enroll, open a lesson, mark it complete, and see the course % update.
- [ ] Reopening a lesson resumes at the saved position, including after logout and login.
- [ ] A quiz can be submitted; score, pass/fail and explanations shown come only from the server response.
- [ ] Home shows Continue Learning, overall progress and streak for a returning user, and an empty state for a new user.
- [ ] Every data screen shows loading, empty, error and offline states.
- [ ] Logout clears all tokens and cached personal data.
- [ ] Account deletion can be requested from Settings.
- [ ] No secrets in the bundle (verified by a build scan).
- [ ] E2E tests for Flows A–D pass.
- [ ] No crash-level errors in a 1-week internal test with at least 10 testers.

* [ ] Every screen, message, notification and lesson shows only Bangla text (except code inside code blocks), with Bengali digits and correct conjunct rendering on low-end Android and iPhone.

## 15. Milestones and open questions

Milestones run in order; each ends with a working build on test devices. Durations are estimates for one developer and should be adjusted to the team.

1. **M1 — Foundation (\~1 week):** Expo project, routing skeleton, theme + core components, API client with generated types, environments.
2. **M2 — Auth + onboarding (\~1 week):** register, login, refresh, forgot password, level/goal, profile. Needs backend auth ready.
3. **M3 — Learning core (\~2 weeks):** roadmap, catalog, course details, enroll, lesson rendering, completion, resume.
4. **M4 — Quizzes + code MCQs (\~1 week):** quiz flow, results, attempt history.
5. **M5 — Dashboard + progress (\~1 week):** Home, My Learning, Progress, streak.
6. **M6 — Hardening + release (\~1–2 weeks):** offline states, accessibility, E2E tests, store listing, internal testing.
7. **Phase 2:** AI Tutor, push reminders, search, video, analytics charts.

**Open questions**

- [ ] App name, logo, primary brand color?
- [ ] Bangla glossary: final Bangla spelling for each technical term (e.g. নিউরাল নেটওয়ার্ক, রিগ্রেশন)?
- [ ] Google sign-in at launch?
- [ ] Free only, or premium plans later (affects store setup and payments)?
- [ ] Analytics tool: PostHog, Firebase, or backend events?
- [ ] Who writes the first course content, and how many courses at launch (suggest: Stage 1–2 complete)?
- [ ] Minimum OS versions after choosing the Expo SDK?
