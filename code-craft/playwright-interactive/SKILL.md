---
name: "playwright-interactive"
description: "Persistent browser and Electron interaction through `js_repl` for fast iterative UI debugging."
---

_Created: 04-07-2026 · Last updated: 29-08-2026_

# Playwright Interactive Skill

Use a persistent `js_repl` Playwright session to debug local web or Electron apps, keep the same handles alive across iterations, and run functional plus visual QA without restarting the whole toolchain unless the process ownership changed.

## Preconditions

- `js_repl` must be enabled for this skill.
- If `js_repl` is missing, enable it in `~/.codex/config.toml`:

```toml
[features]
js_repl = true
```

- You can also start a new session with `--enable js_repl` (equivalent to `-c features.js_repl=true`).
- After enabling `js_repl`, start a new Codex session so the tool list refreshes.
- For now, run this workflow with sandboxing disabled: start Codex with `--sandbox danger-full-access` (or the equivalent config for `sandbox_mode=danger-full-access`). This is a temporary requirement while `js_repl` + Playwright support inside the sandbox is still being completed.
- Run setup from the same project directory you need to debug.
- Treat `js_repl_reset` as a recovery tool, not routine cleanup. Resetting the kernel destroys your Playwright handles.

## One-time setup

```bash
test -f package.json || npm init -y
npm install playwright
# Web-only, for headed Chromium or mobile emulation:
# npx playwright install chromium
# Electron-only, and only if the target workspace is the app itself:
# npm install --save-dev electron
node -e "import('playwright').then(() => console.log('playwright import ok')).catch((error) => { console.error(error); process.exit(1); })"
```

If you switch to a different workspace later, repeat setup there.

## Core Workflow

1. Write a brief QA inventory before testing:
   - Build the inventory from three sources: the user's requested requirements, the user-visible features or behaviors you actually implemented, and the claims you expect to make in the final response.
   - Anything that appears in any of those three sources must map to at least one QA check before signoff.
   - List the user-visible claims you intend to sign off on.
   - List every meaningful user-facing control, mode switch, or implemented interactive behavior.
   - List the state changes or view changes each control or implemented behavior can cause.
   - Use this as the shared coverage list for both functional QA and visual QA.
   - For each claim or control-state pair, note the intended functional check, the specific state where the visual check must happen, and the evidence you expect to capture.
   - If a requirement is visually central but subjective, convert it into an observable QA check instead of leaving it implicit.
   - Add at least 2 exploratory or off-happy-path scenarios that could expose fragile behavior.
2. Run the bootstrap cell once.
3. Start or confirm any required dev server in a persistent TTY session.
4. Launch the correct runtime and keep reusing the same Playwright handles.
5. After each code change, reload for renderer-only changes or relaunch for main-process/startup changes.
6. Run functional QA with normal user input.
7. Run a separate visual QA pass.
8. Verify viewport fit and capture the screenshots needed to support your claims.
9. Clean up the Playwright session only when the task is actually finished.

## Choose Session Mode

For web apps, use an explicit viewport by default and treat native-window mode as a separate validation pass.

- Use an explicit viewport for routine iteration, breakpoint checks, reproducible screenshots, snapshot diffs, and model-assisted localization. This is the default because it is stable across machines and avoids host window-manager variability.
- When you need deterministic high-DPI behavior, keep the explicit viewport and add `deviceScaleFactor` rather than switching straight to native-window mode.
- Use native-window mode (`viewport: null`) for a separate headed pass when you need to validate launched window size, OS-level DPI behavior, browser chrome interactions, or bugs that may depend on the host display configuration.
- For Electron, assume native-window behavior all the time. Electron launches through Playwright with `noDefaultViewport`, so treat it like a real desktop window and check the as-launched size and layout before resizing anything.
- When signoff depends on both layout breakpoints and real desktop behavior, do both passes: explicit viewport first for deterministic QA, then native-window validation for final environment-specific checks.
- Treat switching modes as a context reset. Do not reuse a viewport-emulated `context` for a native-window pass or vice versa; close the old `page` and `context`, then create a new one for the new mode.

## Checklists

### Session Loop

- Bootstrap `js_repl` once, then keep the same Playwright handles alive across iterations.
- Launch the target runtime from the current workspace.
- Make the code change.
- Reload or relaunch using the correct path for that change.
- Update the shared QA inventory if exploration reveals an additional control, state, or visible claim.
- Re-run functional QA.
- Re-run visual QA.
- Capture final artifacts only after the current state is the one you are evaluating.

### Reload Decision

- Renderer-only change: reload the existing page or Electron window.
- Main-process, preload, or startup change: relaunch Electron.
- New uncertainty about process ownership or startup code: relaunch instead of guessing.

### Functional QA

- Use real user controls for signoff: keyboard, mouse, click, touch, or equivalent Playwright input APIs.
- Verify at least one end-to-end critical flow.
- Confirm the visible result of that flow, not just internal state.
- For realtime or animation-heavy apps, verify behavior under actual interaction timing.
- Work through the shared QA inventory rather than ad hoc spot checks.
- Cover every obvious visible control at least once before signoff, not only the main happy path.
- For reversible controls or stateful toggles in the inventory, test the full cycle: initial state, changed state, and return to the initial state.
- After the scripted checks pass, do a short exploratory pass using normal input for 30-90 seconds instead of following only the intended path.
- If the exploratory pass reveals a new state, control, or claim, add it to the shared QA inventory and cover it before signoff.
- `page.evaluate(...)` and `electronApp.evaluate(...)` may inspect or stage state, but they do not count as signoff input.

### Visual QA

- Treat visual QA as separate from functional QA.
- Use the same shared QA inventory defined before testing and updated during QA; do not start visual coverage from a different implicit list.
- Restate the user-visible claims and verify each one explicitly; do not assume a functional pass proves a visual claim.
- A user-visible claim is not signed off until it has been inspected in the specific state where it is meant to be perceived.
- Inspect the initial viewport before scrolling.
- Confirm that the initial view visibly supports the interface's primary claims; if a core promised element is not clearly perceptible there, treat that as a bug.
- Inspect all required visible regions, not just the main interaction surface.
- Inspect the states and modes already enumerated in the shared QA inventory, including at least one meaningful post-interaction state when the task is interactive.
- If motion or transitions are part of the experience, inspect at least one in-transition state in addition to the settled endpoints.
- If labels, overlays, annotations, guides, or highlights are meant to track changing content, verify that relationship after the relevant state change.
- For dynamic or interaction-dependent visuals, inspect long enough to judge stability, layering, and readability; do not rely on a single screenshot for signoff.
- For interfaces that can become denser after loading or interaction, inspect the densest realistic state you can reach during QA, not only the empty, loading, or collapsed state.
- If the product has a defined minimum supported viewport or window size, run a separate visual QA pass there; otherwise, choose a smaller but still realistic size and inspect it explicitly.
- Distinguish presence from implementation: if an intended affordance is technically there but not clearly perceptible because of weak contrast, occlusion, clipping, or instability, treat that as a visual failure.
- If any required visible region is clipped, cut off, obscured, or pushed outside the viewport in the state you are evaluating, treat that as a bug even if page-level scroll metrics appear acceptable.
- Look for clipping, overflow, distortion, layout imbalance, inconsistent spacing, alignment problems, illegible text, weak contrast, broken layering, and awkward motion states.
- Judge aesthetic quality as well as correctness. The UI should feel intentional, coherent, and visually pleasing for the task.
- Prefer viewport screenshots for signoff. Use full-page captures only as secondary debugging artifacts, and capture a focused screenshot when a region needs closer inspection.
- If motion makes a screenshot ambiguous, wait briefly for the UI to settle, then capture the image you are actually evaluating.
- Before signoff, explicitly ask: what visible part of this interface have I not yet inspected closely?
- Before signoff, explicitly ask: what visible defect would most likely embarrass this result if the user looked closely?

### Signoff

- The functional path passed with normal user input.
- Coverage is explicit against the shared QA inventory: note which requirements, implemented features, controls, states, and claims were exercised, and call out any intentional exclusions.
- The visual QA pass covered the whole relevant interface.
- Each user-visible claim has a matching visual check and reviewed screenshot artifact from the state and viewport or window size where that claim matters.
- The viewport-fit checks passed for the intended initial view and any required minimum supported viewport or window size.
- If the product launches in a window, the as-launched size, placement, and initial layout were checked before any manual resize or repositioning.
- The UI is not just functional; it is visually coherent and not aesthetically weak for the task.
- Functional correctness, viewport fit, and visual quality must each pass on their own; one does not imply the others.
- A short exploratory pass was completed for interactive products, and the response mentions what that pass covered.
- If screenshot review and numeric checks disagreed at any point, the discrepancy was investigated before signoff; visible clipping in screenshots is a failure to resolve, not something metrics can overrule.
- Include a brief negative confirmation of the main defect classes you checked for and did not find.
- Cleanup was executed, or you intentionally kept the session alive for further work.

## Dev Server

For local web debugging, keep the app running in a persistent TTY session. Do not rely on one-shot background commands from a short-lived shell.

Use the project's normal start command, for example:

```bash
npm start
```

Before `page.goto(...)`, verify the chosen port is listening and the app responds.

For Electron debugging, launch the app from `js_repl` through `_electron.launch(...)` so the same session owns the process. If the Electron renderer depends on a separate dev server (for example Vite or Next), keep that server running in a persistent TTY session and then relaunch or reload the Electron app from `js_repl`.

## Cleanup

Only run cleanup when the task is actually finished:

- This cleanup is manual. Exiting Codex, closing the terminal, or losing the `js_repl` session does not implicitly run `electronApp.close()`, `context.close()`, or `browser.close()`.
- For Electron specifically, assume the app may keep running if you leave the session without executing the cleanup cell first.

```javascript
if (electronApp) {
  await electronApp.close().catch(() => {});
}

if (mobileContext) {
  await mobileContext.close().catch(() => {});
}

if (context) {
  await context.close().catch(() => {});
}

if (browser) {
  await browser.close().catch(() => {});
}

browser = undefined;
context = undefined;
page = undefined;
mobileContext = undefined;
mobilePage = undefined;
electronApp = undefined;
appWindow = undefined;

console.log("Playwright session closed");
```

If you plan to exit Codex immediately after debugging, run the cleanup cell first and wait for the `"Playwright session closed"` log before quitting.

## Common Failure Modes

- `Cannot find module 'playwright'`: run the one-time setup in the current workspace and verify the import before using `js_repl`.
- Playwright package is installed but the browser executable is missing: run `npx playwright install chromium`.
- `page.goto: net::ERR_CONNECTION_REFUSED`: make sure the dev server is still running in a persistent TTY session, recheck the port, and prefer `http://127.0.0.1:<port>`.
- `electron.launch` hangs, times out, or exits immediately: verify the local `electron` dependency, confirm the `args` target, and make sure any renderer dev server is already running before launch.
- `Identifier has already been declared`: reuse the existing top-level bindings, choose a new name, or wrap the code in `{ ... }`. Use `js_repl_reset` only when the kernel is genuinely stuck.
- `browserContext.newPage: Protocol error (Target.createTarget): Not supported` while working with Electron: do not use `appWindow.context().newPage()` or `electronApp.context().newPage()` as a scratch page; use the Electron-specific screenshot normalization flow in the model-bound screenshots section.
- `js_repl` timed out or reset: rerun the bootstrap cell and recreate the session with shorter, more focused cells.
- Browser launch or network operations fail immediately: confirm the session was started with `--sandbox danger-full-access` and restart that way if needed.

_Dr. Mārcis Gasūns_
## References

Content split (H5588) — sections below were moved verbatim, read on demand:

- [references/bootstrap.md](references/bootstrap.md) — Bootstrap cell and shared web helpers (run once)
- [references/session-recipes.md](references/session-recipes.md) — Desktop/mobile/native-window web and Electron session cells plus iteration reload recipes
- [references/screenshot-examples.md](references/screenshot-examples.md) — Screenshot normalization helpers (web and Electron) and raw-capture exceptions
- [references/viewport-fit-checks.md](references/viewport-fit-checks.md) — Required viewport-fit verification rules and numeric check snippets
