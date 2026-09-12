# NextGig Project Documentation

## Architecture Overview

NextGig is a full-stack MVP built for the SIH 2026 hackathon. It uses a modern Next.js 14 stack (App Router) with React, TypeScript, and Tailwind CSS on the frontend, and robust Next.js API routes on the backend for heavy processing tasks like OCR and AI integrations. The design system leverages Shadcn UI and Framer Motion for premium, animated interfaces.

### Core Stack
- **Framework:** Next.js (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS + CSS Variables (`globals.css`)
- **Components:** Shadcn UI (accessible, unstyled primitives wrapped in Tailwind)
- **Animations:** Framer Motion
- **State Management:** React Context API + LocalStorage/SessionStorage (for persistence without a DB)

### Key Directories
- `app/`: Next.js App Router structure. Contains all pages, layouts, and API routes.
  - `app/api/`: Contains full-stack API routes handling real server-side workloads, including AI generation, PDF parsing, and OCR processing using Tesseract.
  - `app/onboarding/`: The 5-step student onboarding flow.
  - `app/student/`: The student dashboard and opportunities view.
  - `app/recruiter/`: The recruiter dashboard.
- `components/`: Reusable UI components.
  - `components/ui/`: Shadcn primitive components.
  - `components/layout/`: Layout wrappers like the Sidebar.
  - `components/shared/`: Shared components like ThemeToggle, SkillMeter, StatCard, OpportunityCard, etc.
- `lib/`: Business logic, types, and utilities.
  - `lib/ai.ts`: The provider-agnostic wrapper for OpenAI-compatible LLMs. Handles all AI generation tasks.
  - `lib/matching.ts`: The deterministic scoring engine. Calculates match percentages without relying on AI.
  - `lib/data.ts`: Mock data for the MVP (Opportunities, Students).
  - `lib/types.ts`: TypeScript interfaces for the entire app.
- `docs/`: This folder, containing detailed documentation on how the app works.

## How State is Managed

While the application features a robust server-side backend for complex processing (OCR, PDF extraction, AI interaction), we currently use browser storage instead of a traditional database to persist state for the MVP:

1. **SessionStorage:** Used for temporary data during the onboarding flow (e.g., parsed resume data, generated questions, and grade results). This ensures if the user refreshes during onboarding, they don't lose their immediate place.
2. **LocalStorage:** Used for long-term persistence across the app (e.g., the student's completed profile, role selection, and theme preference).
3. **Context API:** 
   - `RoleContext` (`lib/role-context.tsx`): Tracks whether the current user is a 'student' or 'recruiter'.
   - `StudentContext` (`lib/student-context.tsx`): Provides the student's profile data to the dashboard and other views.
   - `ThemeContext` (`lib/theme-context.tsx`): Handles light/dark/recruiter-dark mode switching.

## How the Matching Engine Works (`lib/matching.ts`)

The user requested a strict separation of concerns: AI is for text extraction and chat, but a deterministic engine is the source of truth for match percentages.

The `calculateMatchScore` function uses a weighted algorithm:
- **Skill Match (60%):** Compares the candidate's skill levels against the opportunity's required skills.
- **Education Match (15%):** Checks if the domain/degree aligns.
- **Experience Match (10%):** A simple heuristic based on the number of projects.
- **Verification Bonus (15%):** Rewards candidates who have verified their skills (either via AI assessment or professor endorsement).

## How the AI Integration Works (`lib/ai.ts`)

We use a provider-agnostic wrapper designed to hit any OpenAI-compatible endpoint. You can switch between OpenAI, Anthropic, or an open-source model simply by changing the environment variables in `.env.local`.

- `AI_API_KEY`: Your API key.
- `AI_BASE_URL`: The endpoint URL (default is `https://api.openai.com/v1`).
- `AI_MODEL`: The model name (e.g., `gpt-4o`, `gpt-3.5-turbo`).

The AI handles:
1. Parsing unstructured CV text into a structured JSON profile.
2. Generating a dynamic, 3-question assessment based on the user's claimed skills.
3. Evaluating the subjective answers and updating the user's skill levels.
4. Providing actionable gap analysis and recommendations.

## Future Integration (Insforge)
This full-stack application is built to be easily wired into the broader "Insforge" ecosystem later. The API boundaries are clean and compartmentalized, meaning the existing robust Next.js API routes can act as a standalone microservice or easily connect to a centralized database once it's introduced.
