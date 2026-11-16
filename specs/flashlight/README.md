# Flashlight: hand-off pack

*Plain English: everything Claude Code needs to know about what to build and how it should behave.*

`Feature: Flashlight | Platform: mobile app (iOS + Android) | Version: 1 | Date: 2026-11-16 | Status: draft`

## Summary
A deliberately tiny flashlight app for iOS and Android. One large button turns the phone's light on and off. If the phone can't use its light (no torch, permission refused, too hot or busy), the app offers a full-screen white "screen light" instead. The app always shows the light's true state, shows no ads, needs no account and collects no data. v1 is also a test run of the product-to-build pipeline, so it is kept to the bare minimum.

## Files
| File | What it holds |
|---|---|
| `01-prd.md` | Why we're building it, for whom, scope, constraints, risks and how we'll know it worked |
| `02-requirements.md` | User stories and pass/fail acceptance criteria, plus performance, accessibility and privacy requirements |
| `03-ux-flows.md` | Screens, every screen state, flows, failure flows, microcopy and platform notes |
| `04-tech-spec.md` | *Added by Claude Code:* stack, architecture, error handling, testing |
| `05-tasks.md` | *Added by Claude Code:* the build as vertical slices, each task with a pass/fail check |

## How to build from this
1. Open Claude Code at the project root, in plan mode.
2. Read this whole folder.
3. Write `04-tech-spec.md` and `05-tasks.md` (the `tech-spec-planner` skill does this), as vertical slices, and get them approved.
4. Build one slice at a time. A task is done only when the checks named in its acceptance criteria pass.
5. Resolve the open questions in `01-prd.md` (OQ1–OQ3) during the tech spec; they need a quick device test, not a product decision.

## ID conventions
JS = job story · US = user story · AC = acceptance criterion · NFR = non-functional requirement · SCR = screen · FL = flow · FF = failure flow · EC = edge case · OQ = open question
