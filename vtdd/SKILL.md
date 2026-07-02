---
name: vtdd
description: Use when implementing or reviewing UI from Figma/design screenshots with Visual TDD. Requires full-fidelity restoration of the provided design and visual reference, Figma node inspection when available, failing visual/metric tests before implementation, module-level coverage for every visible functional submodule/form/state, a preflight and user-approval gate before browser visual testing, minimal runtime screenshot/DOM comparison, and explicit reporting of uncovered visual gaps.
---

# Visual TDD

Use this skill when the user asks for visual TDD, pixel-level restoration, Figma-to-code implementation, screenshot comparison, UI visual regression, or says the current page must match a supplied design image or Figma node.

The goal is not "roughly similar." The goal is to turn the design into measurable contracts, watch those contracts fail, implement, and verify the rendered page against both the design source and the user's visual screenshot.

## Core Rule

Do not claim visual fidelity from a passing broad test. Visual TDD must cover every visible structural unit that affects layout, spacing, typography, color, and state, and every visible functional submodule that affects whether the screen is complete.

Pixel fidelity is necessary but not sufficient. A visually close shell is still incomplete if a visible module, form variant, drawer, action row, field group, or state shown by the design/user screenshot is missing from tests or implementation.

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

### 4. Build A Module Inventory

Before writing tests, list every functional submodule visible or implied by the design in the target scope. Treat this as a separate checklist from pixel metrics.

Include, at minimum:

- Each section/card/panel and its intended role.
- Each form module and all visible fields, labels, values, placeholders, validation/help text, and action affordances.
- Each mode or variant shown or implied by visible data, such as corporate/personal payment method, bank/card/crypto channel, default/disabled/pending status, login/register/verification mode, expanded/collapsed rows, and empty/error/loading states.
- Each drawer, modal, popover, upload widget, table row, repeated item, and nested editor opened by a visible action.
- Each interactive control's expected behavior at the smallest useful level: button opens which module, select changes which fields, delete affects which row, verification action sends which channel.

Write this inventory in the working notes or test descriptions. If a module is intentionally out of scope, name it and get explicit user acceptance. Do not silently omit a visible module because the screenshot focus appears to be layout.

### 5. Write Failing Tests First

Before production code changes, add tests that fail for the current implementation.

Prefer a layered test set:

- **Static contract tests** for CSS/module values that map directly to Figma tokens.
- **Component DOM tests** for required structure, semantic labels, and shared component reuse.
- **Module completeness tests** for every visible submodule/form/state from the module inventory. These tests must prove that each field group, action, drawer/modal, row variant, and business branch exists and is wired to the right component.
- **Runtime visual metric tests** with browser/Playwright when possible: bounding boxes, computed styles, relative positions, full-card height.
- **Screenshot comparison** when the project has an image-diff harness. If no harness exists, capture screenshot evidence and compare measured DOM rectangles.

Tests must cover the smallest visible units, not only broad containers. Do not stop after testing card/title/input/button if the design has special rows such as "send code," "forgot password," suffix icons, checkbox, or agreement text.

For forms, test every visible field group and every visible or design-implied variant. Example: if a payment section can render corporate and personal account modules, a passing test for only the card shell and one account type is insufficient.

### 6. Confirm Red

Run the new tests before implementation.

The failure must be meaningful:

- Missing node/class for a Figma element.
- Wrong size, line height, gap, padding, color, or state.
- Wrong structure such as label text and action link sharing the wrong row height.

If a test passes immediately, it either covers existing correct behavior or is too weak. Strengthen it or add a runtime measurement.

### 7. Implement Minimal Green

Change production code only after the red test is proven.

Implementation rules:

- Reuse existing project patterns and styling approach.
- Keep scope to the target module.
- Prefer semantic structure that mirrors Figma's grouping when grouping affects layout.
- Give compound rows their own class or component when their metrics differ from ordinary fields.
- Preserve business behavior while changing visual structure.
- Do not hide mismatches by changing tests to match current code.

### 8. Prepare Browser Visual Testing

Use browser visual testing sparingly and deliberately. Before opening or controlling a browser, prepare and report the preflight conditions:

- Dev server command, host, port, and whether an existing service is being reused.
- Target route, viewport(s), locale, theme, feature flags, and domain/host requirements.
- Auth/session plan, test account or mocked state, and the exact fixture data needed to render each module variant.
- Browser tooling status: Playwright/browser binary or Chrome path, screenshot output path, and any known sandbox or permission limitations.
- Minimal browser action plan: pages to open, clicks/inputs needed, screenshots to capture, and DOM metrics to read.

First inspect the implementation yourself with non-browser evidence: Figma metadata, code, unit/DOM tests, and static screenshots if available. Do not use browser automation as the first discovery tool.

Before AI operates a browser for visual testing, pause and tell the user:

- What has already been checked without a browser.
- The prepared environment and exact route/viewports/actions.
- What screenshots or measurements will be collected.
- Any account/data/sandbox risk.

Wait for explicit user approval before continuing. If the user does not approve, stop before browser control and report the unverified visual areas.

### 9. Runtime Compare Against Figma

After tests pass and the user approves browser visual testing, open the actual route in a browser and measure the rendered UI.

For each important Figma node, compare:

- Relative `x/y` inside the card or page.
- `width/height`.
- `font-size`, `line-height`, `font-weight`, `color`.
- `background`, `border-radius`, padding.
- Gaps between adjacent elements.
- Visual state such as disabled/default colors.

Use exact values where possible. Allow only small browser text-rendering variance when unavoidable, and name the tolerance.

Keep browser work minimal:

- Prefer one route load per viewport.
- Prefer computed DOM metrics over repeated manual visual probing.
- Capture targeted screenshots of the module and one full viewport only when needed.
- Avoid broad exploratory clicking; use the module inventory to drive only required interactions.

### 10. Screenshot Review

Capture the rendered app screenshot at the same viewport as the user's screenshot when possible.

Compare visually and numerically:

- Full card dimensions and center position.
- Top, middle, and bottom rhythm.
- Special rows: verification code, forgot password, password suffix icons.
- Bottom agreement alignment and checkbox size.
- Header/background only if in scope.

If the browser host blocks the intended domain, use an equivalent route only if it renders the exact same component, and state that substitution.

### 11. Report Honestly

Final or status reports must include:

- Figma node id(s) used.
- What pixel/metric tests and module completeness tests were red and then green.
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
- Implementing a visually similar section while omitting form variants, drawers, repeated rows, or business submodules visible in the design.
- Running browser visual testing without first preparing route/auth/data/browser conditions and pausing for user approval.
- Using browser automation as broad exploration when a smaller DOM/unit/component test would catch the issue.
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

## Context From A Later Supplier Basic Info Miss

A later miss happened on the Tiansuan supplier basic information page. The visual shell of the "收款方式" card looked close, but the payment method form module was incomplete.

The key lesson:

- A card-level visual contract is not enough when the card contains business-specific subforms.
- The module inventory must include the payment method form branches, such as corporate account, personal account, bank card/channel fields, and any drawer opened by "修改".
- Tests must prove both the display grid and the edit form modules exist. A test that only checks card height, title, edit button, and generic value boxes can still pass while the actual form is missing.
- Browser visual testing should be prepared as a final confirmation step, not used to discover that a module was omitted. The omission should be caught by module completeness tests first.

## Completion Checklist

Before finishing, verify:

- Figma context and metadata were read, or unavailability was explicitly stated.
- Visual inventory includes all visible nodes in scope.
- Module inventory includes every visible functional submodule, form branch, drawer/modal, repeated item, and state in scope.
- Tests failed before implementation.
- Tests pass after implementation.
- Browser visual testing preflight was prepared, reported to the user, and explicitly approved before any browser control.
- Browser runtime measurements are minimal and cover disputed and high-risk areas.
- Screenshot evidence was captured or the reason it was not captured is stated.
- Final response lists exact verification commands, module coverage, and residual risk.
