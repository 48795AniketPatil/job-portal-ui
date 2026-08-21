# AGENTS.md

## Project

This repository is a Vite + React job portal UI. The current implementation uses JavaScript/JSX, Tailwind CSS, React Context, React Router, mock data, simulated services, and browser `localStorage`. It does not currently use TypeScript or Supabase.

Primary areas:

- `src/pages/`: route-level screens
- `src/components/`: shared UI
- `src/context/` and `src/contexts/`: shared state and data-fetching contexts
- `src/services/`: simulated API services
- `src/data/mockData.js`: seed data

## Instruction Priority

1. Explicit user requests for the current task
2. The nearest applicable `AGENTS.md`
3. `general.instructions.md`, `copilot-instructions.md`, `typescript-react.instructions.md`, and `design.instructions.md`
4. `CLAUDE.md` and existing repository conventions
5. Agent defaults

When instructions conflict, follow the higher-priority source and preserve the narrowest change that satisfies the request. Treat the referenced instruction files as the source of truth for their respective topics; do not duplicate their detailed rules here.

## Workflow

- Inspect the relevant page, component, context, service, and nearby tests before editing.
- Make small, focused changes that follow existing patterns and preserve public behavior.
- Avoid broad refactors, unrelated formatting, dependency upgrades, and speculative abstractions.
- Keep data access in services and shared state in contexts. Preserve state and `localStorage` consistency.
- Add or update focused tests when the changed behavior has test coverage; do not hide or weaken failures.
- Run the narrowest relevant validation first, then the broader checks available below.
- Report changed files, behavior changes, validation commands, and any known limitations.

## Commands

Run these from the repository root:

- Install dependencies: `npm install`
- Start development server: `npm run dev`
- Production build: `npm run build`
- Lint: `npm run lint`
- Preview production build: `npm run preview`

The current `package.json` does not define `format`, `typecheck`, or `test` scripts. Do not add or claim those checks pass unless the project is updated to provide them. If these scripts are introduced, use `npm run format`, `npm run typecheck`, and `npm run test` as the standard commands.

## Security Boundaries

- Never commit secrets, tokens, credentials, or populated environment files.
- Do not weaken TypeScript strictness if TypeScript is introduced or migrated.
- Do not bypass authentication, role checks, protected routes, or authorization logic.
- Treat client-side mock authentication and `localStorage` as demo-only; do not represent them as secure server-side controls.
- Validate untrusted input and avoid introducing unsafe HTML or code execution.
- Do not expose private user or application data through shared state, URLs, logs, or errors.

## Files Not To Modify

Do not modify `dist/`, lockfiles, generated files, build output, or dependency artifacts unless the user explicitly requests it. Do not manually edit generated configuration or tool output.

## Future Stack Migration

If the project is migrated to Vite + React + TypeScript + Supabase, update this file and the relevant instruction files together. At that point, strict TypeScript and the established Supabase service/auth boundaries become mandatory; components must not bypass those services with direct database access.
