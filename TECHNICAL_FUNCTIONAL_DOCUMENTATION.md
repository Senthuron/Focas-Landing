## CA Guru.AI Landing – Technical & Functional Documentation

---

### 1. High‑Level System Architecture

**Purpose**  
This project is a **single‑page marketing & download funnel** for the **CA Guru.AI** mobile application (by FOCAS Edu). It is optimized for:

- Presenting marketing content (hero, features, testimonial, FAQ).
- Driving users to download the CA Guru.AI app.
- Tracking download events in a remote database (Supabase).

**Architecture Overview**

- **Client‑side only web app**:
  - Built with **React 18 + TypeScript**.
  - Bundled by **Vite** and served as static assets (HTML, JS, CSS, images).
  - No custom backend service in this repository.
- **Routing**:
  - Implemented with **React Router DOM**:
    - `/` – Main landing page (`Index.tsx`).
    - `/download-success` – Post‑download confirmation page.
    - `*` – 404 / Not Found page.
- **State / data providers**:
  - **React Query** (`QueryClientProvider`) is configured globally for future data‑fetching; currently the key runtime integration is Supabase tracking via a custom hook.
  - **TooltipProvider** and two toast/toaster systems (`Toaster` and `Sonner`) provide UI feedback and notifications.
- **Data & integrations**:
  - **Supabase (PostgreSQL as a service)** for download counter tracking:
    - Frontend directly communicates with Supabase via the JS client.
    - No intermediate API layer.
  - **Google Drive** (or other APK host) for app download link.
- **UI / Styling**:
  - **Tailwind CSS** with custom design tokens in `index.css`.
  - **shadcn/ui** (Radix‑based components) under `src/components/ui`.
  - Custom components such as `NotificationBar` and layout sections in `Index.tsx`.

The result is a **pure frontend, static‑host deployable** application with **one remote integration (Supabase)** for telemetry and a remote file host for the APK.

---

### 2. Folder and File Structure Explanation

> Note: Only key directories and files are documented here; some utility/config files are omitted for brevity.

#### Root

- `package.json`  
  - Defines dependencies and NPM scripts:
    - `dev`, `build`, `build:dev`, `lint`, `preview`.
- `vite.config.ts`  
  - Vite configuration:
    - Dev server on port `8080`, host `::` (all interfaces).
    - `@` path alias → `./src`.
    - React SWC plugin and `lovable-tagger` plugin in development.
- `tsconfig.json`  
  - TypeScript compiler options and path aliases (`@/*` → `src/*`).
- `tailwind.config.ts`, `postcss.config.js`  
  - Tailwind and PostCSS setup.
- `eslint.config.js`  
  - ESLint configuration for React, TypeScript, and hooks.
- `components.json`  
  - shadcn/ui configuration.
- `index.html`  
  - Vite root HTML template; mounting point for React.
- `README.md`  
  - General project overview (user‑focused).
- `TECHNICAL_FUNCTIONAL_DOCUMENTATION.md`  
  - This file: deep technical and functional documentation.

#### `/src`

- `main.tsx`
  - Vite entry file; mounts `<App />` to the DOM.
- `App.tsx`
  - **Composition root**:
    - Creates a `QueryClient` and wraps the app in `QueryClientProvider`.
    - Wraps UI with `TooltipProvider`.
    - Renders two toaster systems:
      - `Toaster` from `@/components/ui/toaster`.
      - `Sonner` from `@/components/ui/sonner`.
    - Configures `BrowserRouter` with three `Route`s:
      - `/` → `Index` (landing page).
      - `/download-success` → `DownloadSuccess`.
      - `*` → `NotFound`.

##### `/src/pages`

- `Index.tsx`
  - **Main landing page** and primary functional core of the app.
  - Responsibilities:
    - Renders `NotificationBar` at the top.
    - Manages local state:
      - `isNotificationVisible`: controls layout padding vs notification visibility.
      - `isMobileMenuOpen`: placeholder for mobile navigation (currently unused in the provided snippet).
    - On mount (`useEffect`):
      - Reads `localStorage` key `ca-guru-notification-dismissed` to decide whether to show the notification.
    - Defines an FAQ JSON‑LD object (`faqJson`) and injects it via a `<script type="application/ld+json">` tag for SEO.
    - Layout sections:
      - **Header & Navigation**:
        - Mobile and desktop variants.
        - Links to external site `https://focasedu.com`.
        - CTAs bound to `downloadApp` from `useDownload`.
      - **Hero**: main headline, description, feature bullets, and CTA buttons.
      - **Features**: series of cards highlighting core app benefits.
      - **Social Proof**: testimonial section.
      - **Download CTA**: prominent gradient section with download button and supporting text.
      - **FAQ**: collapsible `<details>` elements with common questions.
      - **Footer**: copyright notice with dynamic year.
- `DownloadSuccess.tsx`
  - Simple confirmation page rendered after a user completes the download funnel (reachable via router).
  - Typically includes:
    - A success message.
    - A link/button to download again and/or return to home.
- `NotFound.tsx`
  - Catch‑all 404 page for undefined routes.

##### `/src/components`

- `NotificationBar.tsx`
  - Top‑of‑page banner used for promotions, announcements, or messages.
  - Expected behavior:
    - Can be dismissed by the user.
    - Writes `ca-guru-notification-dismissed` to `localStorage` when dismissed.
    - On subsequent visits, `Index.tsx` uses this key to control whether the bar is visible.
- `ui/`
  - Generated **shadcn/ui** components based on Radix primitives:
    - `button.tsx`, `input.tsx`, `dialog.tsx`, `tooltip.tsx`, `dropdown-menu.tsx`, etc.
  - Used extensively in the landing page for consistent styling and behavior.
  - UI components include styling hooks into Tailwind via className props and `class-variance-authority` variants.

##### `/src/hooks`

- `useDownload.ts`
  - **Core business logic hook** for the download funnel.
  - State:
    - `isDownloading: boolean` – prevents duplicate clicks or simultaneous download flows.
  - Functions:
    - `incrementCounter`:
      - Uses Supabase RPC to increment a download counter:
        - Calls `(supabase as any).rpc('execute_sql', { query: '...' })`.
        - Raw SQL performs:
          - `INSERT INTO "download-counter" (id, count) VALUES (1, 1)`
          - `ON CONFLICT (id)` → `UPDATE` count and `updated_at`.
      - Logs to console on error but does not block the user flow.
    - `downloadApp`:
      - Guard: if `isDownloading` is true, immediately returns.
      - Sets `isDownloading` to true.
      - Calls `incrementCounter()` and then:
        - Uses `toast` from `@/hooks/use-toast` to show:
          - **Success**: “Download started – Opening app download page...”
        - Opens the APK link in a new tab:
          - `https://drive.google.com/file/d/10kL_arM8x6FlfWG3OGqMOv2KY_7CYzoQ/view?usp=drive_link`
      - On error:
        - Logs error to console.
        - Shows a destructive toast: “Download failed – Please try again later.”
      - In `finally`, resets `isDownloading` to false.
  - Return value:
    - `{ downloadApp, isDownloading }` used across the UI (e.g., buttons in `Index.tsx`).
- `use-mobile.tsx`
  - Helper for mobile behavior (e.g., tracking viewport, mobile‑specific logic or UI).
  - Not directly central to the core download flow.
- `use-toast.ts`
  - Wiring into the toast system used across the app:
    - Provides `toast` function used in `useDownload`.
    - Likely wraps Radix toast or `sonner` to expose a simple API.

##### `/src/integrations/supabase`

- `client.ts`
  - **Supabase client factory**:
    - Uses `createClient<Database>` from `@supabase/supabase-js`.
    - Hard‑coded:
      - `SUPABASE_URL = "https://xbsvyxubnjdionidncau.supabase.co"`.
      - `SUPABASE_PUBLISHABLE_KEY = "<anon key>"`.
    - Client config:
      - `auth.storage = localStorage`.
      - `persistSession = true`, `autoRefreshToken = true`.
  - Export:
    - `export const supabase = createClient<Database>(...)`.
  - Used primarily by `useDownload.ts`.
- `types.ts`
  - TS type definitions for the Supabase database schema (`Database`).
  - Enables type‑safe operations against the Supabase client.

##### `/src/lib`

- `utils.ts`
  - Utility helpers (e.g., `cn` function that merges class names).

##### `/src/index.css`

- Global styles including:
  - Tailwind base, components, and utilities.
  - CSS custom properties for colors, radii, etc.
  - Custom classes such as `bg-gradient-primary`, `shadow-elevated`, animations like `animate-floaty`.

#### `/public`

- Static assets:
  - `/lovable-uploads/*.png` – hero and feature imagery.
  - `focas-logo.png` – logo used in header.
  - `placeholder.svg`, `robots.txt`, etc.

#### `/supabase`

- `config.toml`  
  - Supabase project configuration.
- `migrations/*.sql`
  - SQL migration(s) defining tables such as `"download-counter"` used for tracking.

---

### 3. Core Logic Areas

Although this project is primarily a marketing/landing site (and not a full application with user accounts or dashboards), it still has several important logical components.

#### 3.1 Authentication

- **There is currently no user authentication in this repository.**
  - Supabase is used **only** as an anonymous client for logging download counts.
  - No sign‑up, login, or session‑based features are implemented.
  - Supabase client is configured with `auth` options (e.g., `persistSession: true`), but these are not actively used for a user‑facing auth flow.

If authentication is added in the future, typical changes would include:

- Extending the Supabase configuration to handle sign‑in/sign‑out.
- Adding protected routes in React Router.
- Surfacing session status in UI components (e.g., user menu).

#### 3.2 Download Funnel & Telemetry

**Main logic location**: `src/hooks/useDownload.ts` and its usage in `src/pages/Index.tsx`.

Flow:

1. User clicks a “Get App” or “Download Free” button.
2. `downloadApp` is invoked:
   - If a download is already in progress (`isDownloading === true`), the call is ignored.
   - Sets `isDownloading = true` to lock the button.
3. `incrementCounter` sends a raw SQL statement via Supabase RPC (`execute_sql`) to:
   - Insert / upsert a row into `"download-counter"` table with `id = 1`.
   - Increment `count` on conflict and update `updated_at`.
4. On successful increment (or even if it fails silently), UI flow:
   - Shows informational toast: “Download started – Opening app download page...”.
   - Opens `apkUrl` in a new browser tab (currently a Google Drive link).
5. On error:
   - Logs error to console.
   - Shows destructive toast: “Download failed – Please try again later.”
6. `isDownloading = false` is restored to re‑enable the button.

**Functional Impact**:

- Ensures a consistent count of download attempts in Supabase for analytics.
- Prevents rapid re‑clicks from spamming the database.
- Provides user feedback via toasts.

#### 3.3 Notification System

**Location**: `NotificationBar.tsx`, `Index.tsx`, `use-toast.ts`, `App.tsx`.

- **Toasts**:
  - `App.tsx` renders:
    - `Toaster` – likely shadcn/ui toast system.
    - `Sonner` – alternative toast system (via `@/components/ui/sonner`).
  - `use-toast.ts` exposes a `toast` API used by `useDownload` to show success/error messages.
- **Top Notification Banner**:
  - `NotificationBar` likely includes:
    - A message and CTA (e.g., a promotion).
    - A “dismiss” action.
  - Dismissal writes `ca-guru-notification-dismissed` into `localStorage`.
  - `Index.tsx`:
    - On mount, reads that key and sets `isNotificationVisible` accordingly.
    - Adjusts header padding to account for the banner’s height.

#### 3.4 SEO & FAQ Integration

**Location**: `Index.tsx`.

- `faqJson` object:
  - Represented as:
    - `@context: "https://schema.org"`.
    - `@type: "FAQPage"`.
    - `mainEntity`: array of Q/A pairs describing key FAQs.
- Injected into a `<script type="application/ld+json">` with `dangerouslySetInnerHTML`.
- Helps search engines treat the FAQ content as structured data for rich results.

#### 3.5 UI Architecture & Design System

- **Layout & Sections**:
  - Entire landing layout is composed in `Index.tsx` using Tailwind utility classes.
  - Reusable patterns:
    - Container: `container mx-auto px-4`.
    - Reusable sections: hero, features, social proof, download CTA, FAQ, footer.
- **Components**:
  - `Button` from `@/components/ui/button` with variants:
    - `variant="cta"` – primary marketing CTAs.
    - `variant="secondary"` – secondary actions (e.g., “See how it works”).
  - Other shadcn/ui components available but not heavily used in the current snippet.

---

### 4. Deployment and Rollback Procedure

Since this is a **static‑asset Vite app** with a remote Supabase backend, deployment is straightforward. Below is a generic approach you can adapt to platforms like Vercel, Netlify, Render, or any static host.

#### 4.1 Build & Deploy

1. **Install dependencies**
   ```bash
   npm install
   ```

2. **Build the project**
   ```bash
   npm run build
   ```
   - This creates an optimized production build in the `dist/` directory.

3. **Upload / deploy `dist/`**
   - For static hosts:
     - Configure the host to serve `dist/` as the web root.
   - For Vercel:
     - Framework preset: **Vite**.
     - Build command: `npm run build`.
     - Output directory: `dist`.
   - For Netlify:
     - Build command: `npm run build`.
     - Publish directory: `dist`.

4. **Environment variables (optional but recommended)**
   - If you refactor Supabase keys into environment variables:
     - Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` in your hosting provider.
   - Update `client.ts` to use `import.meta.env.VITE_SUPABASE_URL`, etc.

5. **Supabase configuration**
   - Ensure the Supabase project referenced in `client.ts`:
     - Exists and is configured.
     - Has a `download-counter` table matching the expected schema.
     - Has appropriate RLS policies to allow the increment operation from anonymous clients.

#### 4.2 Rollback Procedure

Because the app is static:

- **Application rollback**:
  - Revert to a previous build artifact:
    - Many static hosts (Netlify, Vercel) provide UI to “rollback” to a prior deployment with a single click.
  - Alternatively:
    - Re‑deploy a previously known‑good `dist/` build or git commit.
      ```bash
      # Example:
      git checkout <previous-stable-commit>
      npm install
      npm run build
      # Deploy resulting dist/ as before
      ```

- **Supabase / database rollback**:
  - If a migration or table change causes issues:
    - Revert the migration using Supabase migration tooling or manually.
    - Restore from a Supabase backup or point‑in‑time recovery if necessary.
  - Because this app performs only **simple increment operations** on `download-counter`, risks of schema rollback are modest, but misconfigured RLS or renamed tables can break telemetry.

#### 4.3 Smoke Testing After Deploy/Rollback

After deploying or rolling back:

1. Visit the site root `/`:
   - Check hero, navigation, CTA buttons.
   - Verify that clicking “Get App” opens the APK link in a new tab.
2. Confirm that toasts appear on download click.
3. Confirm `/download-success` route loads as expected (if linked from the UI).
4. In Supabase dashboard:
   - Verify that `"download-counter"` table receives increments after test clicks.

---

### 5. Known Risks or Technical Debt

#### 5.1 Hard‑coded Supabase Credentials

- **Issue**:
  - `SUPABASE_URL` and `SUPABASE_PUBLISHABLE_KEY` are currently hard‑coded in `src/integrations/supabase/client.ts`.
  - While the anon key is meant to be public, embedding it directly in code:
    - Makes environment‑specific configuration harder.
    - Encourages accidental reuse across staging/production.
- **Mitigation**:
  - Move these values to environment variables (`VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`).
  - Configure per‑environment values in the hosting platform.

#### 5.2 Raw SQL RPC (`execute_sql`)

- **Issue**:
  - `useDownload.ts` calls an RPC function `execute_sql` with raw SQL to increment the counter.
  - This approach:
    - Is less type‑safe and less explicit than using a defined Postgres function or standard Supabase table operations.
    - Can become brittle if table names/columns change.
    - Relies on Supabase having a custom function named `execute_sql` exposed to the client.
- **Mitigation**:
  - Replace raw SQL RPC with:
    - A dedicated function in Postgres (e.g., `increment_download_counter()`) exposed via Supabase RPC.
    - Or a simple `supabase.from("download-counter").upsert(...)` call.

#### 5.3 Anonymous, Unthrottled Download Increments

- **Issue**:
  - Any anonymous client can trigger `incrementCounter` repeatedly.
  - This may inflate download statistics if the endpoint is abused (e.g., bots).
- **Mitigation**:
  - Implement rate limiting or basic protections:
    - RLS policies that impose some constraints (e.g., IP logging and thresholds, if feasible).
    - Frontend mitigation (e.g., `isDownloading` already rate‑limits per click session, but not across sessions).
    - Consider moving counting behind a server‑side endpoint with rate limiting.

#### 5.4 Missing Auth & User Context

- **Issue**:
  - There is no concept of an authenticated user or per‑user behavior.
  - All analytics are per‑download only.
- **Mitigation / Future work**:
  - If needed, introduce optional auth to:
    - Track downloads by user segment.
    - Provide custom experiences (e.g., returning user messaging).

#### 5.5 UX/Content Coupling in `Index.tsx`

- **Issue**:
  - Most of the page’s content and structure lives in a single `Index.tsx` file.
  - As the page grows, this can:
    - Make maintenance harder.
    - Discourage reuse of sections.
- **Mitigation**:
  - Extract major sections into separate components:
    - `HeroSection`, `FeaturesSection`, `TestimonialsSection`, `DownloadSection`, `FaqSection`, etc.
  - This will improve readability and enable reuse across future pages.

#### 5.6 Dual Toast Systems

- **Issue**:
  - Both `Toaster` and `Sonner` are rendered in `App.tsx`.
  - This can cause confusion over which toast system should be used.
- **Mitigation**:
  - Standardize on one toast system.
  - Remove unused one from `App.tsx` and align `use-toast.ts` implementation accordingly.

---

### 6. Summary

This repository implements a **focused, static React landing page** with:

- Minimal but meaningful logic for **download tracking via Supabase**.
- A clear conversion funnel from visit → interest → download.
- An extensible scaffolding (React Query, shadcn/ui, Supabase types) that can support future features such as more complex forms, analytics, or authenticated dashboards.

Future work should primarily focus on **hardening integrations (Supabase, analytics)**, **decoupling content into reusable components**, and **moving configuration into environment variables** to improve maintainability and security.

