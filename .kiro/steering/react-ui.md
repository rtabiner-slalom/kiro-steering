---
inclusion: fileMatch
fileMatchPattern: ['ui/**', 'demo_api/**']
---

# React Demo UI

The UI exists to show the agent working. Keep it small, clear and safe. Do not gold-plate it, and do not cut corners on correctness or safety.

## Stack

- React function components and hooks, Vite, TypeScript with `strict: true`, plain CSS files.
- No state libraries, no UI component frameworks, and no CSS-in-JS unless the user asks.
- Inline style objects only for values that are dynamic.

## Structure

- `ui/src/api/` holds a typed client for the demo API. Define request and response types in one place. If the API is described by an OpenAPI document, generate the types from it.
- Components are small and named in `PascalCase`. Keep one component per file.
- Configuration (API base URL) comes from `import.meta.env` with a `VITE_` prefix. No secrets in the UI.

## Chat behaviour

- Show the session's messages, an input box, and a clear sending state.
- Always show the project's demo banner (for example "Synthetic data. Not for real decisions."), defined in `product.md`.
- Show tool activity or citations when the API returns them, so reviewers can see where an answer came from.
- Handle and display these states: loading, empty, network error, API error, timeout. Never leave the user with a silent failure.
- Disable the send button while a request is in flight. Do not allow double submission.
- Keep the session ID in memory or `sessionStorage`. Offer a "new conversation" action.
- Stream responses only if the API supports it and the design asks for it.

## Safety

- Render model output as text. Do not use `dangerouslySetInnerHTML`. If markdown rendering is required, use a sanitising renderer and say so in the design.
- Do not trust any value from the API for navigation or script execution.
- Do not log message content to the console in committed code.

## Accessibility

- Use semantic elements and labelled inputs. Make the message list an `aria-live` region so new replies are announced.
- Everything works with the keyboard alone. Keep focus visible.
- Meet basic colour contrast.

## Checks

- Prettier, ESLint with `typescript-eslint`, and `tsc --noEmit` pass. See `coding-standards.md`.
- Test the main flow and the error states with Vitest and React Testing Library. See `testing.md`.
- The dev server runs on the origin that `DEMO_UI_ORIGIN` allows. Moving the UI to S3 and CloudFront means updating CORS and adding authentication first.
