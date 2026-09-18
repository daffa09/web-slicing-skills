# web-slicing-skills — agent instructions

Plain-markdown version of the two skills in this repo, with no Claude-specific
frontmatter. [AGENTS.md](https://agents.md) is read automatically by most coding
agents (OpenAI Codex, Cursor, Windsurf, GitHub Copilot Coding Agent, Google Gemini CLI,
Aider, Zed, and others) when it lives at the root of a repo. To use these skills in your
own project with a non-Claude agent, copy the relevant section below into that project's
own `AGENTS.md`, `.cursor/rules/`, `.windsurfrules`, or equivalent instructions file.

Trigger: the user asks to slice, clone, or match a UI 1:1 against a mockup (a URL, an
HTML file, an image, or a design tool like Figma).

Playwright is required for both skills (measuring the source and generating overlay
diffs). Install it in a scratch folder outside the target repo, not as a project
dependency — see rule 5 in each section. Claude Code users installing this as a plugin
get Playwright pre-installed automatically; every other agent installs it on first use
as described below.

---

## Slicing web UI 1:1

### Input

| Field | Required | Content / default |
|---|---|---|
| Source | yes | URL, path to `.html` file/mockup folder, path to an image, or MCP + frame link |
| Target | yes | route or file path (may not exist yet) |
| Scope | no | one full page |
| Viewport | no | 1440x900; mobile web: 390x844; image source: image width ÷ scale (2880px @2x → 1440) |
| Notes | no | required components, how to log in, etc. |

- Source or Target missing: ask for both in one message, naming the source type, any
  connected design MCP, and the optional fields' defaults. Wait for the answer.
- Design MCP not connected: offer to connect it, or use a different source.
- Image pasted without a path: ask for the path (pixel sampling needs the file).
- Everything given: restate Input as one block, then proceed without further
  confirmation.

### Rules

1. Copy exactly. No redesigning, no swapped text, no added or removed elements.
2. Every value comes from a measurement (computed style / MCP data / pixel sampling),
   never a guess.
3. Code/MCP source: use precise values, don't round to a design token; note anything
   outside the token set. Image source: round to the nearest token, note anything off
   by more than 2px.
4. Use the project's existing components and tokens. If a shared component's size
   differs from the source, don't change the component, override it locally, and note
   it.
5. Don't add a dependency to the target project. Install Playwright in a scratch folder
   outside the repo if it isn't already available. Source fonts can be added the
   framework's normal way; mention it in the report.
6. Dummy data must match the source exactly (text, item counts).
7. Only the Viewport given in Input, no breakpoints or states absent from the source.
8. No inline banner/section-divider comments and no comments that just restate the
   code. Comment only for a reason that isn't obvious from the code itself.
9. Only stop to ask if the source can't be opened, or the target needs a login with no
   way to mock it.

### Steps

**0. Learn the project.** Framework, styling approach (Tailwind / CSS modules /
etc.), token/theme file, existing components, folder conventions, how to run it. Start
the dev server and confirm the target route opens.

**1. Measure the source.** Work in a scratch folder outside the repo. Required
artifacts: `ref-full.png` (1x, width = Viewport), a screenshot per section, and a spec.
- **URL / file**: use Playwright at the given Viewport (mobile web:
  `isMobile: true`, `hasTouch: true`, `deviceScaleFactor: 1`). Wait for
  `document.fonts.ready` and network idle, disable animations/transitions. `file://`
  doesn't render correctly — serve the folder instead (e.g. `npx serve`). Dump computed
  style for every visible element to `ref-styles.json`: selector, text (≤ 40 chars),
  bounding box, font-family/size/weight, line-height, letter-spacing, color,
  background, padding, margin, gap, display + flex/grid, border, border-radius,
  box-shadow, opacity. Check hover/focus states on interactive elements with `hover()` /
  `focus()`.
- **Image**: scale = image width ÷ Viewport width; produce a 1x version. Get spacing
  and color from pixel sampling (`canvas.getImageData` via Playwright). Identify fonts
  from letterform shape; if unsure between two fonts, render both and compare.
- **MCP**: screenshot the frame at 1x, capture layout/style values (size, auto layout,
  gap, font, color, radius, shadow) and variables/tokens into `ref-styles.json`. Any
  code the MCP generates is a source of numbers only, never copy it directly.
- Spec: fonts, tokens (color, font scale, spacing, radius, shadow), frame (container,
  gutter, grid, header/sidebar height), and a top-to-bottom list of sections.

**2. Build section by section.** Order: fonts → tokens → frame → sections. For each
section:
1. Implement it, then screenshot the target at the same Viewport.
2. Compare:
   - Visual: overlay `ref-full.png` on top of the target build (data URL,
     `position:absolute; top:0; left:0; pointer-events:none;
     mix-blend-mode:difference`), screenshot it, and look at the image. Black means it
     matches, bright means it's off.
   - Numbers (code/MCP source): compare computed style of the same element against
     `ref-styles.json`.
3. Repeat until position/size differ by ≤ 1px, font/color/radius/shadow match, and the
   overlay is just faint noise at text edges. Max 5 rounds; if it's still off, note why
   and move on.

Finish with a full-page comparison.

Most common misses: line-height, letter-spacing, font-weight 600 vs 700, a border that
adds to box size, icon size/stroke, shadow, and container max-width/gutter.

**3. Report (short).** Files created/changed. A table per section: status, remaining
diff, reason. Values outside the token set; shared components overridden locally. Paths
to the source, result, and last overlay screenshots.

---

## Slicing mobile UI 1:1

Covers Flutter, React Native/Expo, native Android, and native iOS.

### Input

| Field | Required | Content / default |
|---|---|---|
| Source | yes | URL, path to `.html` file/mockup folder, path to an image, or MCP + frame link |
| Target | yes | screen name, route, or file path (may not exist yet) |
| Scope | no | one full screen |
| Device | no | the source's logical size (e.g. 390x844 / 360x800); @2x/@3x images divided by their scale |
| Platform | no | whatever is available on this machine (iOS simulator is macOS-only) |
| Notes | no | required components, how to log in, etc. |

Same intake rules as the web version above: ask once if Source or Target is missing,
offer to connect a design MCP if it isn't, ask for a path if an image was pasted
without one, otherwise restate Input and proceed.

### Rules

1. Copy exactly. No redesigning, no swapped text, no added or removed elements.
2. Every value comes from a measurement (computed style / MCP data / pixel sampling),
   never a guess. Use logical units (dp / pt), not physical pixels.
3. Code/MCP source: use precise values, don't round to a token; note anything outside
   the token set. Image source: round to the nearest token, note anything off by more
   than 2dp.
4. Use the project's existing components, theme, and tokens. If a shared component's
   size differs from the source, don't change the component, override it locally, and
   note it.
5. Don't add a dependency. Install Playwright (for measuring the source and doing the
   overlay) in a scratch folder outside the repo if it isn't already available. Source
   fonts can be bundled the framework's normal way (pubspec fonts, expo-font,
   `res/font`, Info.plist); mention it in the report.
6. Dummy data must match the source exactly. Note any temporary change (dummy data,
   initial route, deep link) in the report.
7. Only the device size and orientation from the source, no tablet layout, landscape,
   or dark mode absent from the source.
8. Don't hardcode the status bar / safe area — use the platform API (`SafeArea`,
   `useSafeAreaInsets`, `WindowInsets`, safe-area-inset), matching the source visually.
9. Don't change the physical phone's settings. Only change size/density on an
   emulator, and reset it afterward.
10. No inline banner/section-divider comments and no comments that just restate the
    code. Comment only for a reason that isn't obvious from the code itself.
11. Only stop to ask if the source can't be fetched, no emulator/simulator can run, or
    the target screen needs a login with no way to mock it.

### Steps

**0. Learn the project and device.** Framework (Flutter / React Native / Expo /
Compose or XML / SwiftUI or UIKit), theme/token setup, existing components, navigation
(how to open the target screen directly), how to run it.
- Device: `adb devices`, `emulator -list-avds`, `flutter devices`, or
  `xcrun simctl list devices`. If nothing is running, start one.
- If the logical size differs from the Device field (Android emulator):
  `adb shell wm size <px>` + `adb shell wm density <dpi>` (px = dp × dpi ÷ 160). Reset
  with `adb shell wm size reset`, `adb shell wm density reset`.
- Match the status bar (Android demo mode, or
  `xcrun simctl status_bar booted override --time 9:41` on iOS), or crop it from both
  images.
- Run the app and open the target screen.

**1. Measure the source.** Work in a scratch folder outside the repo. Required
artifacts: `ref-full.png` (1x, width = the Device's logical width), a screenshot per
section, and a spec.
- **URL / file**: Playwright at the Device's viewport, `isMobile: true`,
  `hasTouch: true`, `deviceScaleFactor: 1` (1 CSS px = 1 dp/pt). Wait for
  `document.fonts.ready` and network idle, disable animations/transitions. `file://`
  doesn't render correctly, serve the folder instead. Dump computed style for every
  visible element to `ref-styles.json` (same fields as the web version above).
- **Image**: scale = image width ÷ the Device's logical width; produce a 1x version.
  Spacing and color from pixel sampling. Identify fonts from letterform shape.
- **MCP**: screenshot the frame at 1x, capture layout/style values and
  variables/tokens into `ref-styles.json`. MCP-generated code is a source of numbers
  only.
- Spec: fonts (already bundled or not), tokens, frame (status bar / safe area, app bar
  height, bottom nav/tab bar height, horizontal padding), and a top-to-bottom section
  list.

**2. Build section by section.** Order: fonts → tokens → frame → sections. For each
section:
1. Implement it, make sure the app updates (hot reload/fast refresh/rebuild), then
   screenshot it:
   - Android: `adb exec-out screencap -p > target.png`
   - iOS: `xcrun simctl io booted screenshot target.png`
   - Flutter, faster: a golden test at the same size and device pixel ratio, loading
     the real fonts first (the test harness's default fonts aren't the app's fonts).
   - A mobile MCP (e.g. mobile-mcp) can be used for screenshots/navigation if
     connected.
   - Screen longer than the viewport: screenshot per scroll position against the
     matching part of the source.
2. Compare:
   - Visual: stack `ref-full.png` and the target screenshot in a simple HTML page, both
     at the Device's logical CSS width, top image with `mix-blend-mode:difference`.
     Render with Playwright, screenshot it, and look at the image.
   - Numbers (where possible): Android native / React Native via
     `adb shell uiautomator dump`, bounds in px ÷ (dpi ÷ 160) = dp. Flutter: DevTools
     layout explorer or `debugDumpRenderTree()`. Compare against `ref-styles.json`.
3. Repeat until position/size differ by ≤ 1dp, font/color/radius/shadow match, and the
   overlay is just faint noise at text edges. Max 5 rounds; if it's still off, note why
   and move on.

Finish with a full-screen comparison.

Most common misses:
- Line-height: React Native Android `includeFontPadding`, Flutter `height` +
  `TextHeightBehavior`.
- A font weight that isn't bundled falls back to a different weight.
- Shadow: Android `elevation` doesn't match a design shadow.
- Safe area, status bar, and default app-bar/tab-bar height from the navigation
  library.
- Minimum touch target size adding to an element's size (Material 48dp).

**3. Report (short).** Files created/changed. Platform and device checked (other
platforms may render fonts slightly differently). A table per section: status,
remaining diff, reason. Values outside the token set; shared components overridden
locally. Any temporary change still in place, and whether the emulator was reset. Paths
to the source, result, and last overlay screenshots.
