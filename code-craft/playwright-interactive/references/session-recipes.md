## Start or Reuse Web Session

Desktop and mobile web sessions share the same `browser`, helpers, and QA flow. The main difference is which context and page pair you create.

### Desktop Web Context

Set `TARGET_URL` to the app you are debugging. For local servers, prefer `127.0.0.1` over `localhost`.

```javascript
var TARGET_URL = "http://127.0.0.1:3000";

if (page?.isClosed()) page = undefined;

await ensureWebBrowser();
context ??= await browser.newContext({
  viewport: { width: 1600, height: 900 },
});
page ??= await context.newPage();

await page.goto(TARGET_URL, { waitUntil: "domcontentloaded" });
console.log("Loaded:", await page.title());
```

If `context` or `page` is stale, set `context = page = undefined` and rerun the cell.

### Mobile Web Context

Reuse `TARGET_URL` when it already exists; otherwise set a mobile target directly.

```javascript
var MOBILE_TARGET_URL = typeof TARGET_URL === "string"
  ? TARGET_URL
  : "http://127.0.0.1:3000";

if (mobilePage?.isClosed()) mobilePage = undefined;

await ensureWebBrowser();
mobileContext ??= await browser.newContext({
  viewport: { width: 390, height: 844 },
  isMobile: true,
  hasTouch: true,
});
mobilePage ??= await mobileContext.newPage();

await mobilePage.goto(MOBILE_TARGET_URL, { waitUntil: "domcontentloaded" });
console.log("Loaded mobile:", await mobilePage.title());
```

If `mobileContext` or `mobilePage` is stale, set `mobileContext = mobilePage = undefined` and rerun the cell.

### Native-Window Web Pass

```javascript
var TARGET_URL = "http://127.0.0.1:3000";

await ensureWebBrowser();

await page?.close().catch(() => {});
await context?.close().catch(() => {});
page = undefined;
context = undefined;

browser ??= await chromium.launch({ headless: false });
context = await browser.newContext({ viewport: null });
page = await context.newPage();

await page.goto(TARGET_URL, { waitUntil: "domcontentloaded" });
console.log("Loaded native window:", await page.title());
```

## Start or Reuse Electron Session

Set `ELECTRON_ENTRY` to `.` when the current workspace is the Electron app and `package.json` points `main` to the right entry file. If you need to target a specific main-process file directly, use a path such as `./main.js` instead.

```javascript
var ELECTRON_ENTRY = ".";

if (appWindow?.isClosed()) appWindow = undefined;

if (!appWindow && electronApp) {
  await electronApp.close().catch(() => {});
  electronApp = undefined;
}

electronApp ??= await electronLauncher.launch({
  args: [ELECTRON_ENTRY],
});

appWindow ??= await electronApp.firstWindow();

console.log("Loaded Electron window:", await appWindow.title());
```

If `js_repl` is not already running from the Electron app workspace, pass `cwd` explicitly when launching.

If the app process looks stale, set `electronApp = appWindow = undefined` and rerun the cell.

If you already have an Electron session but need a fresh process after a main-process, preload, or startup change, use the restart cell in the next section instead of rerunning this one.

## Reuse Sessions During Iteration

Keep the same session alive whenever you can.

Web renderer reload:

```javascript
await reloadWebContexts();
```

Electron renderer-only reload:

```javascript
await appWindow.reload({ waitUntil: "domcontentloaded" });
console.log("Reloaded Electron window");
```

Electron restart after main-process, preload, or startup changes:

```javascript
await electronApp.close().catch(() => {});
electronApp = undefined;
appWindow = undefined;

electronApp = await electronLauncher.launch({
  args: [ELECTRON_ENTRY],
});

appWindow = await electronApp.firstWindow();
console.log("Relaunched Electron window:", await appWindow.title());
```

If your launch requires an explicit `cwd`, include the same `cwd` here.

Default posture:

- Keep each `js_repl` cell short and focused on one interaction burst.
- Reuse the existing top-level bindings (`browser`, `context`, `page`, `electronApp`, `appWindow`) instead of redeclaring them.
- If you need isolation, create a new page or a new context inside the same browser.
- For Electron, use `electronApp.evaluate(...)` only for main-process inspection or purpose-built diagnostics.
- Fix helper mistakes in place; do not reset the REPL unless the kernel is actually broken.

