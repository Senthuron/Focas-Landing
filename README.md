## CA Guru.AI – FOCAS Landing

Marketing landing page for **CA Guru.AI**, an AI‑powered mentor app for CA Foundation, Intermediate, and Final students, built for high‑conversion downloads and fast iteration. The site is a **React + Vite + TypeScript** single‑page app using **Tailwind CSS** and **shadcn/ui** components, with **Supabase** used to track app downloads.

---

### Features

- **High‑converting hero**: Clear headline, value proposition, and strong CTAs to download the app or explore features.
- **Feature highlights**: Cards explaining instant doubt solving, CA‑specific training, 24/7 mentor support, and more.
- **Social proof**: Testimonial section to build trust and credibility.
- **Download funnel**:
  - Primary CTAs throughout the page (hero, sticky navigation, download section).
  - Download button triggers Supabase logging and opens the app’s APK link.
  - Dedicated `/download-success` page.
- **FAQ section**:
  - Expandable questions for common objections.
  - JSON‑LD `FAQPage` structured data for SEO.
- **Notification bar**:
  - Dismissible banner (e.g. announcements) with `localStorage` persistence.
- **Responsive layout**:
  - Optimized for mobile‑first viewing.
  - Separate mobile and desktop navigation layouts.
- **SEO & sharing**:
  - Meta tags and structured data for improved visibility (FAQ and software/application schema).
  - `robots.txt` included.

---

### Tech Stack

- **Frontend**: React 18 + TypeScript
- **Build tool**: Vite 5 (React SWC plugin)
- **Routing**: React Router DOM
- **Styling**:
  - Tailwind CSS with custom design tokens in `index.css`.
  - Utility helpers such as `cn` in `src/lib/utils.ts`.
- **UI components**:
  - shadcn/ui (built on Radix UI primitives) in `src/components/ui`.
  - Custom `NotificationBar` component.
- **State & data**:
  - TanStack React Query (for data fetching/caching; available for future use).
  - React Hook Form + Zod (form + validation, ready for contact / waitlist forms).
- **Backend / data layer**:
  - Supabase JavaScript client for remote PostgreSQL.
- **Tooling**:
  - ESLint for linting.
  - TypeScript for type checking.
  - PostCSS + Tailwind for styling.

---

### Project Structure

High‑level layout:

- `src/`
  - `main.tsx` – App entrypoint that mounts React.
  - `App.tsx` – Root component that wires up routing and providers (e.g. React Router, React Query).
  - `pages/`
    - `Index.tsx` – Main landing page at `/` (hero, features, social proof, download CTA, FAQ).
    - `DownloadSuccess.tsx` – Confirmation page after a successful download flow.
    - `NotFound.tsx` – 404 page for unknown routes.
  - `components/`
    - `ui/` – shadcn/ui components (buttons, dialogs, inputs, etc.).
    - `NotificationBar.tsx` – Dismissible, persistent top‑of‑page notification.
  - `hooks/`
    - `useDownload.ts` – Encapsulates download‑tracking logic (Supabase write + opening APK link).
    - `use-mobile.tsx` – Mobile‑specific UI behavior/helper.
    - `use-toast.ts` – Toast/notification hook (via Sonner/shadcn patterns).
  - `integrations/supabase/`
    - `client.ts` – Supabase client configuration.
    - `types.ts` – Generated Supabase database types.
  - `lib/utils.ts` – Utility helpers (e.g. `cn` class name merger).
  - `index.css` – Global styles, Tailwind base, and design tokens.
- `public/`
  - `lovable-uploads/` – Landing page imagery and screenshots.
  - `focas-logo.png`, `placeholder.svg`, `robots.txt`, etc.
- `supabase/`
  - `config.toml` – Supabase project configuration.
  - `migrations/` – SQL migrations (e.g. download counter table).
- Root configuration:
  - `package.json`, `vite.config.ts`, `tsconfig.json`, `tailwind.config.ts`,
    `postcss.config.js`, `eslint.config.js`, `components.json`, `index.html`.

---

### Routing

Routing is handled with **React Router DOM**:

- `/` – Main CA Guru.AI marketing / download page (`Index.tsx`).
- `/download-success` – Download confirmation and secondary CTA.
- `*` – Catch‑all 404 page.

The router is configured in `App.tsx` using `<BrowserRouter>` with nested `<Routes>` and `<Route>` components.

---

### Download Tracking & Supabase Integration

The project integrates with **Supabase** to track app downloads:

- **Client setup**:
  - `src/integrations/supabase/client.ts` initializes a Supabase client with a project URL and anon key.
- **Database**:
  - A `download-counter` table (see `supabase/migrations/...sql`) stores download counts.
- **Hook**:
  - `useDownload`:
    - Handles the click on “Get app” / “Download Free”.
    - Sends an update to Supabase to increment the download counter.
    - Optionally shows toast notifications to confirm success/failure.
    - Opens the actual APK link (hosted on Google Drive) in a new tab.

> **Note:** In production, ensure that Supabase has proper **Row Level Security (RLS)** rules and that public updates are safe and rate‑limited. This repo is optimized for a simple marketing funnel rather than complex access control.

---

### Notification Bar Behavior

The `NotificationBar` at the top of the page is:

- **Dismissible** – Users can close it.
- **Persistent** – Dismissal is stored in `localStorage` with the key `ca-guru-notification-dismissed`, so it does not reappear on subsequent visits from the same browser.

The landing page header (`Index.tsx`) checks this key on mount and adjusts layout padding when the notification is visible.

---

### SEO & Structured Data

The landing page includes:

- **FAQ structured data**:
  - `faqJson` object in `Index.tsx` rendered via a `<script type="application/ld+json">` tag.
  - Helps Google and other search engines understand the FAQ content.
- **Marketing copy tuned for CA students**:
  - Headline, body, and feature bullets tailored to CA Foundation, Inter, and Final.
- **robots.txt**:
  - Stored in `public/robots.txt` to control crawler behavior.

You can further extend SEO by adding meta tags and Open Graph tags in `index.html` or via additional components.

---

### Getting Started

#### Prerequisites

- **Node.js** (LTS recommended) and **npm**.

#### Installation

```bash
npm install
```

This installs both runtime and development dependencies defined in `package.json`.

#### Development Server

```bash
npm run dev
```

- Starts Vite’s dev server (configured to run on port **8080** by default).
- Visit `http://localhost:8080` in your browser.

Hot Module Replacement (HMR) is enabled for rapid iteration.

#### Building for Production

```bash
# Standard production build
npm run build

# Development-mode build (uses the `development` mode)
npm run build:dev
```

Build artifacts are output to the `dist/` directory and can be served by any static host.

#### Previewing the Production Build

```bash
npm run preview
```

This runs a local server that serves the contents of `dist/`, mimicking a production environment.

---

### Environment & Configuration

Currently, Supabase credentials are **embedded in the code** under `src/integrations/supabase/client.ts`. For a real production deployment, you should:

- Move sensitive values into environment variables.
- Pass them to the client using Vite’s `import.meta.env` pattern (e.g. `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`).
- Ensure only public, non‑secret keys are exposed on the frontend.

Example (conceptual only):

```ts
const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;
```

Then set these in your hosting platform’s environment configuration.

---

### Running Supabase Locally (Optional)

The codebase expects a **hosted Supabase** instance and does not require Supabase to run locally. However, the `supabase/` directory contains configuration and migration files that you can use if you want to replicate the database schema.

High‑level steps:

1. Create a Supabase project (in the Supabase dashboard).
2. Apply the SQL file(s) from `supabase/migrations/` to your database.
3. Update the Supabase URL and anon key in `src/integrations/supabase/client.ts` (or environment variables).

---

### Scripts (from `package.json`)

- **`npm run dev`** – Start Vite dev server.
- **`npm run build`** – Create a production build.
- **`npm run build:dev`** – Build using development mode.
- **`npm run lint`** – Run ESLint over the project.
- **`npm run preview`** – Preview the production build locally.

---

### Styling & Design System

- **Tailwind CSS**:
  - Configured in `tailwind.config.ts` with custom colors and utilities.
  - Dark mode is class‑based (`class` strategy).
  - Animations (e.g. `accordion`, floaty hero image) configured via `tailwindcss-animate`.
- **Design tokens**:
  - Defined via CSS variables in `index.css` for colors, border radius, and other primitives.
- **Components**:
  - shadcn/ui components live under `src/components/ui`.
  - Buttons use variants such as `variant="cta"` and `variant="secondary"` for primary/secondary actions.

---

### Development Notes

- **TypeScript** is enabled but `strict` mode is relaxed for development velocity.
- **ESLint** configuration (`eslint.config.js`) provides:
  - React, React Hooks, and TypeScript‑aware linting.
  - React Refresh plugin for a smoother dev experience.
- **Lovable integration**:
  - `lovable-tagger` plugin is configured in Vite for component tagging when used on the Lovable platform.

---

### Extending the Project

Some ideas for extending this landing page:

- **Add analytics**:
  - Integrate Google Analytics, Fathom, or another analytics provider to track page views and funnel performance.
- **Add waitlist or contact forms**:
  - Use React Hook Form + Zod to capture email addresses and questions.
  - Store entries in Supabase or a dedicated email provider.
- **A/B test CTAs and copy**:
  - Swap headlines, button labels, and hero imagery to test conversion.
- **Multi‑language support**:
  - Introduce i18n to support different regions.

---

### License / Ownership

This landing page and branding are tailored to **CA Guru.AI** and **FOCAS Edu**. If you intend to reuse the code or branding, ensure you have the appropriate rights and update logos, copy, and assets as needed.

# Welcome to your Lovable project

## Project info

**URL**: https://lovable.dev/projects/6d54553d-da3c-4517-b6af-3951f2dc9600

## How can I edit this code?

There are several ways of editing your application.

**Use Lovable**

Simply visit the [Lovable Project](https://lovable.dev/projects/6d54553d-da3c-4517-b6af-3951f2dc9600) and start prompting.

Changes made via Lovable will be committed automatically to this repo.

**Use your preferred IDE**

If you want to work locally using your own IDE, you can clone this repo and push changes. Pushed changes will also be reflected in Lovable.

The only requirement is having Node.js & npm installed - [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating)

Follow these steps:

```sh
# Step 1: Clone the repository using the project's Git URL.
git clone <YOUR_GIT_URL>

# Step 2: Navigate to the project directory.
cd <YOUR_PROJECT_NAME>

# Step 3: Install the necessary dependencies.
npm i

# Step 4: Start the development server with auto-reloading and an instant preview.
npm run dev
```

**Edit a file directly in GitHub**

- Navigate to the desired file(s).
- Click the "Edit" button (pencil icon) at the top right of the file view.
- Make your changes and commit the changes.

**Use GitHub Codespaces**

- Navigate to the main page of your repository.
- Click on the "Code" button (green button) near the top right.
- Select the "Codespaces" tab.
- Click on "New codespace" to launch a new Codespace environment.
- Edit files directly within the Codespace and commit and push your changes once you're done.

## What technologies are used for this project?

This project is built with:

- Vite
- TypeScript
- React
- shadcn-ui
- Tailwind CSS

## How can I deploy this project?

Simply open [Lovable](https://lovable.dev/projects/6d54553d-da3c-4517-b6af-3951f2dc9600) and click on Share -> Publish.

## Can I connect a custom domain to my Lovable project?

Yes, you can!

To connect a domain, navigate to Project > Settings > Domains and click Connect Domain.

Read more here: [Setting up a custom domain](https://docs.lovable.dev/tips-tricks/custom-domain#step-by-step-guide)
