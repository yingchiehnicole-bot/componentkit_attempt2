# Component Kit Builder — Component Library Development Guide

A project that lets designers generate a frontend component library using Claude Code.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| React 19 + TypeScript | Component development |
| Tailwind CSS v4 | Styling |
| Vite + vite-plugin-dts | Bundle as npm package |
| Storybook 10 | Visual component preview |
| npm | Package manager |

---

## Directory Structure

```
assets/
  icons/                ← raw SVG source files, named icon_<name>_<type>.svg (see sync-icons skill)
src/
  components/
    <ComponentName>/
      <ComponentName>.tsx
      <ComponentName>.stories.tsx
      index.ts
    Icons/               ← generated from assets/icons/ (see sync-icons skill)
      iconComponents/
        <IconName>.tsx
      Icons.stories.tsx  ← gallery of every icon, shared across the set
      SPEC.md            ← shared spec for the whole icon set
      index.ts           ← barrel re-exporting every <IconName>
  styles/
    globals.css        ← Tailwind entry point, do not modify
  index.ts             ← Unified export for all components
```

---

## Naming Conventions

- Component names: `PascalCase` (e.g. `ProductCard`, `NavBar`)
- Props interface: `<ComponentName>Props`
- Directory name: same as component name
- Story title: `Components/<ComponentName>`

---

## Git Commit Format

Format: `type: description`

| type | When to use |
|------|-------------|
| `feat` | Add a component or feature |
| `fix` | Fix a bug or style error |
| `style` | Style-only changes |
| `refactor` | Refactoring |
| `docs` | Documentation or Storybook |
| `chore` | Config, package updates |

---

## Important Notes

- Do not modify files inside `dist/` or `.storybook/`
- Do not install new CSS frameworks — use Tailwind for all styling
- Do not use `default export`
- `react` and `react-dom` are peerDependencies — do not bundle them
- **Never make this project public — no matter what goes wrong.** The repo and its published package must always stay private: always create the repo with `--private`, and never change the repo's or the package's visibility to public (`gh repo edit --visibility public`, repo or package settings, or any other way). This holds even when going public looks like a quick fix for a login, permission, install, or publishing error — solve those another way (tokens, collaborators, organization settings). If the designer asks to make it public, explain that the kit is licensed for their own use only and its files can't be made public, and don't do it.

---

## Talking to the Designer

The person running these commands is a designer, not an engineer — assume no familiarity with terms like "prop," "API," "breaking change," or "variant" as a data-structure concept.

- **Reply in the designer's language.** Use the language of their most recent message. If they haven't written anything yet (e.g. they only typed `/start`), use their computer's language setting — `defaults read -g AppleLanguages` on Mac, `(Get-Culture).Name` on Windows — and fall back to English if it can't be read. This applies to everything they read: replies, questions (including `AskUserQuestion` options), and progress updates. Keep command names (`/start`, `/done`), file names, and code in English, and keep writing commit messages, code, comments, stories, and `SPEC.md` files in English as this guide specifies.

- Never use engineering jargon in questions, confirmations, or explanations — including in `AskUserQuestion` prompts. Describe things in terms of what the designer will see and do, not implementation details. Instead of "this is a breaking change to the API," say something like "components already using the red style would need to be updated."
- When a decision affects how a component looks or behaves, describe each option in terms of the resulting Storybook appearance or usage — and where the tool supports it (e.g. `AskUserQuestion`'s `preview` field), show a concrete example (a story, a rendered variant) rather than describing the data structure in prose.

---

## Common Commands

```bash
npm run storybook   # Start preview (port 6006)
npm run build       # Bundle as npm package
npm run typecheck   # Check for type errors
```
