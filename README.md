English | [Bahasa Indonesia](README.id.md)

# web-slicing-skills

Two Claude Code skills for pixel-perfect UI slicing from a mockup: `/slice-web` for
websites and web apps (including mobile web), `/slice-mobile` for Flutter, React
Native/Expo, native Android, and native iOS. The source can be a URL, an HTML file, an
image, or a design MCP like Figma. Output is checked with Playwright: the screenshot
gets laid over the source, and the diff stays visible until what's left is just faint
noise at the edges of the text.

## Install

**Option 1, plugin (one line, Playwright installs itself):**

```
/plugin marketplace add daffa09/web-slicing-skills
/plugin install web-slicing-skills@web-slicing-skills
```

Skills become `/web-slicing-skills:slice-web` and `/web-slicing-skills:slice-mobile`.
Playwright and Chromium install automatically during the first Claude Code session after
install, not the instant `/plugin install` finishes, but once the next session opens.
Requires Node.js on your machine and one internet download (~150 MB for Chromium).

**Option 2, manual copy (keeps the short names `/slice-web` and `/slice-mobile`):**

```
cp -r skills/slice-web skills/slice-mobile ~/.claude/skills/
```

Playwright isn't automatic on this path. The skill installs it to a temp folder the
first time it's called.

**Option 3, any other AI coding agent (Cursor, Windsurf, Copilot Coding Agent, OpenAI
Codex, Gemini CLI, Aider, Zed, ...):**

This isn't Claude-only. [`AGENTS.md`](AGENTS.md) has the same two skills as plain
markdown, no Claude-specific frontmatter, following the open
[agents.md](https://agents.md) convention most agents already read automatically.
Drop it (or the relevant section) into your own project's `AGENTS.md`,
`.cursor/rules/`, `.windsurfrules`, or whatever instructions file your agent uses.
Playwright still needs installing yourself there, it's a manual step outside a project's
dependencies, same as option 2 above.

## Requirements

- Claude Code.
- Node.js (for Playwright).
- One internet connection upfront to download Chromium.
- Optional: a design MCP (e.g. Figma) if your source is MCP.
- Optional: `graphify` on `PATH` to save tokens finding components/tokens in a large
  Next.js, Flutter, or React Native repo. Works fine without it too, the skill falls
  back to plain grep.

## Usage

```
/slice-web https://example.com/pricing src/app/pricing/page.tsx
/slice-mobile ./mockup/onboarding-3.png OnboardingScreen
```

Leave out the source or target and the skill asks once, then runs to completion without
asking again. Output is a short report: files changed, remaining diff per section, and
the path to the last overlay screenshot.

## What it holds to

- Copy exactly, no redesigning, no swapped text, no added or missing elements.
- Every value comes from measurement (computed style / MCP data / pixel sampling),
  never guessed.
- Reuse the project's existing components and tokens instead of inventing new ones.
- No new dependency in your project. Playwright only runs in a separate folder, outside
  the repo being sliced.

Full rules live in `skills/slice-web/SKILL.md` and `skills/slice-mobile/SKILL.md`.

## Troubleshooting

Chromium failed to download automatically (network dropped during the first session)?
Ask Claude to retry, or install it manually:

```
cd ~/.claude/plugins/data/web-slicing-skills-web-slicing-skills
npx playwright install chromium
```

## License

MIT
