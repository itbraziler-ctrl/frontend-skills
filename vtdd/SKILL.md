---
name: vtdd
description: Use when implementing or reviewing UI from Figma/design screenshots with Visual TDD. Requires full-fidelity restoration of the provided design and visual reference, Figma node inspection when available, failing visual/metric tests before implementation, runtime screenshot/DOM comparison, and explicit reporting of any uncovered visual gaps.
---

# Visual TDD

Use this skill when the user asks for visual TDD, pixel-level restoration, Figma-to-code implementation, screenshot comparison, UI visual regression, or says the current page must match a supplied design image or Figma node.

The goal is not "roughly similar." The goal is to turn the design into measurable contracts, watch those contracts fail, implement, and verify the rendered page against both the design source and the user's visual screenshot.

## Core Rule

Do not claim visual fidelity from a passing broad test. Visual TDD must cover every visible structural unit that affects layout, spacing, typography, color, and state.

If the user provides both Figma and a screenshot:

1. Figma is the source of truth for node metrics and tokens.
2. The screenshot is the source of truth for what the user notices in the current viewport.
3. Browser runtime measurement is the source of truth for what the app actually renders.

All three must be reconciled.

## Required Workflow

### 1. Capture Inputs

Collect and name all visual inputs:

- Figma URL, file key, and node id.
- User screenshots or clipboard images.
- Target route, viewport, host, and browser context.
- Existing page/component files and styling system.

If a Figma URL includes `node-id=21076-12966`, parse it as `21076:12966`.

### 2. Read Figma Before Guessing

Use Figma tools when available:

- `get_design_context` first for generated reference code, screenshot, tokens, and asset hints.
- `get_metadata` for the complete node tree: ids, names, positions, dimensions.
- `get_screenshot` when a direct design render is needed.
- `use_figma` only after loading `figma-use`, and only when metadata/context tools are insufficient.

If Figma tools are unavailable, say so clearly. Do not present screenshot-derived guesses as Figma pixel data.

### 3. Build A Visual Inventory

From Figma metadata, create a checklist of every visible node or component group that affects the final UI. Include, at minimum:

- Page/shell background, header, card/container.
- Every text node: content, font size, line height, weight, color.
- Every field group: label row, control row, helper/action text, suffix icons.
- Every compound row, not just outer controls.
- Gaps between groups and any padding hidden inside parent frames.
- Button states and disabled states if visible in the app.
- Bottom/meta areas such as login hints, agreement text, checkbox, terms links.

For each item, record expected `x`, `y`, `width`, `height`, key styles, and parent-relative position where useful.

### 4. Write Failing Tests First

Before production code changes, add tests that fail for the current implementation.

Prefer a layered test set:

- **Static contract tests** for CSS/module values that map directly to Figma tokens.
- **Component DOM tests** for required structure, semantic labels, and shared component reuse.
- **Runtime visual metric tests** with browser/Playwright when possible: bounding boxes, computed styles, relative positions, full-card height.
- **Screenshot comparison** when the project has an image-diff harness. If no harness exists, capture screenshot evidence and compare measured DOM rectangles.

Tests must cover the smallest visible units, not only broad containers. Do not stop after testing card/title/input/button if the design has special rows such as "send code," "forgot password," suffix icons, checkbox, or agreement text.

### 5. Confirm Red

Run the new tests before implementation.

The failure must be meaningful:

- Missing node/class for a Figma element.
- Wrong size, line height, gap, padding, color, or state.
- Wrong structure such as label text and action link sharing the wrong row height.

If a test passes immediately, it either covers existing correct behavior or is too weak. Strengthen it or add a runtime measurement.

### 6. Implement Minimal Green

Change production code only after the red test is proven.

Implementation rules:

- Reuse existing project patterns and styling approach.
- Keep scope to the target module.
- Prefer semantic structure that mirrors Figma's grouping when grouping affects layout.
- Give compound rows their own class or component when their metrics differ from ordinary fields.
- Preserve business behavior while changing visual structure.
- Do not hide mismatches by changing tests to match current code.

### 7. Runtime Compare Against Figma

After tests pass, open the actual route in a browser and measure the rendered UI.

For each important Figma node, compare:

- Relative `x/y` inside the card or page.
- `width/height`.
- `font-size`, `line-height`, `font-weight`, `color`.
- `background`, `border-radius`, padding.
- Gaps between adjacent elements.
- Visual state such as disabled/default colors.

Use exact values where possible. Allow only small browser text-rendering variance when unavoidable, and name the tolerance.

### 8. Screenshot Review

Capture the rendered app screenshot at the same viewport as the user's screenshot when possible.

Compare visually and numerically:

- Full card dimensions and center position.
- Top, middle, and bottom rhythm.
- Special rows: verification code, forgot password, password suffix icons.
- Bottom agreement alignment and checkbox size.
- Header/background only if in scope.

If the browser host blocks the intended domain, use an equivalent route only if it renders the exact same component, and state that substitution.

### 9. Report Honestly

Final or status reports must include:

- Figma node id(s) used.
- What tests were red and then green.
- Runtime measured values for the highest-risk elements.
- Commands run and pass/fail status.
- Any remaining mismatches, unavailable tools, or unverified areas.

Do not say "pixel-perfect" unless every visible unit in scope has been covered by tests or runtime measurements.

## Anti-Patterns

Avoid these failures:

- Testing only card width, title, first input, and submit button while ignoring compound rows.
- Treating a screenshot crop as Figma source-of-truth when Figma metadata is available.
- Ignoring disabled/default state differences such as a "send code" button turning gray.
- Missing hidden spacing like bottom padding inside a parent frame.
- Claiming a green test means visual fidelity when the test does not inspect the disputed area.
- Updating tests after implementation without first seeing the old implementation fail.
- Comparing only absolute page coordinates when card-relative coordinates are what matters.

## Context From The Original Session

This skill was drafted after a Visual TDD miss on the Tiansuan `fronted` supplier registration page.

The first attempt incorrectly claimed visual parity after testing only broad elements:

- Card shell.
- Title.
- Generic input controls.
- Submit button.

The user correctly pointed out that the differences were still large, especially:

- `邮箱验证码` row.
- `发送验证码` action text.
- `密码 / 忘记密码?` row.
- Confirm-password bottom spacing.
- Login/agreement bottom rhythm.

Root cause:

- Figma metadata showed compound rows (`21033:32009`, `21033:32013`, `21033:32017`, `21033:32020`, `21033:32029`) with their own `32px` header rows, `51px` inputs, and `22px` bottom padding.
- The original tests did not cover those nodes.
- Browser runtime measurement only inspected card/title/first input/submit, so it gave a false sense of completion.

Corrected contract examples from that incident:

- Card `488 x 868`, padding `16px 32px`.
- Form `424 x 696`.
- Title `29.4px / 45px`, title-to-form gap `32px`.
- Ordinary field group `22px label + 8px gap + 51px input = 81px`.
- Verification title row `32px`; verification input `51px`.
- Send-code text `#6016d9`, `14px / 22px`.
- Password title row `32px`; forgot-password action `57 x 32`.
- Confirm-password group `73px` because it includes `22px` bottom padding.
- Invite-code group `91px` because it includes `10px` bottom padding.
- Login hint `14px / 21px` at card-relative `y=801`.
- Agreement row `12px / 18px` at card-relative `y=834`; checkbox `16px`.

When a user says "视觉 TDD" in this project, assume they expect this level of coverage.

## Completion Checklist

Before finishing, verify:

- Figma context and metadata were read, or unavailability was explicitly stated.
- Visual inventory includes all visible nodes in scope.
- Tests failed before implementation.
- Tests pass after implementation.
- Browser runtime measurements cover disputed and high-risk areas.
- Screenshot evidence was captured or the reason it was not captured is stated.
- Final response lists exact verification commands and residual risk.
