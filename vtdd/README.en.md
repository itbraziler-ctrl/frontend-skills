# VTDD

VTDD is a Codex skill for pixel-level frontend UI restoration. It turns Figma designs, user screenshots, and browser runtime rendering into measurable visual contracts, then requires failing tests before implementation and runtime DOM/screenshot verification after implementation.

## When To Use

- The user asks for "Visual TDD" or pixel-level restoration.
- A Figma frame/node must be implemented as a frontend page or component.
- UI visual regression needs screenshot, Playwright, or DOM rect verification.
- The user says the implementation still does not match the design and the work needs a systematic pass over missed nodes, spacing, typography, colors, and states.

## Core Principle

Figma is the source of truth for node metrics and design tokens. The user's screenshot is the source of truth for what they notice in the current viewport. Browser runtime measurement is the source of truth for what the app actually renders. All three need to be reconciled before claiming visual fidelity.

## Workflow

1. Collect the Figma URL, node id, screenshots, target route, viewport, and existing implementation files.
2. Read Figma context, metadata, and screenshot first when available; clearly report if tools are unavailable.
3. Build a complete visual inventory covering all visible nodes, compound rows, hidden padding, states, and bottom/meta areas.
4. Write failing tests before touching production code, covering CSS contracts, DOM structure, runtime dimensions, and screenshot evidence.
5. Implement the smallest production change after proving the red test.
6. Measure the real browser route for card-relative coordinates, dimensions, typography, colors, gaps, padding, and states.
7. Report the Figma nodes used, red/green tests, runtime measurements, command results, and remaining risks.

## Install

Copy `vtdd/` into the global Codex skills directory:

```bash
cp -R vtdd ~/.codex/skills/vtdd
```

Restart Codex after installation so the new skill is loaded.

## Example Prompts

```text
Use vtdd to implement this registration form from the Figma frame with Visual TDD.
```

```text
This page still looks different from the screenshot. Use Visual TDD to compare and fix it fully.
```
