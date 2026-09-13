---
name: csscreme
description: Give a coding agent a real design system before it writes UI. Fetches a DESIGN.md (semantic tokens, measured WCAG contrast, checkable rules) from CSS Crème, or extracts one from any live site or screenshot, then installs the matching shadcn/ui theme. Use whenever the task involves building or restyling UI and no design system is defined - "build a dashboard", "make a settings page", "this looks generic", "match the design of <site>", "pick a color scheme", "set up a theme", "our components look inconsistent", "add dark mode", "check contrast". Also use when a repo has no DESIGN.md and UI work is starting.
license: MIT
---

# csscreme

Free design context for coding agents. No account, no key, no paid tier.

## Why this exists

An agent asked for UI with no design system does the same thing every time: invents a palette,
mixes three or four corner radii, adds a second font nobody asked for, and ships contrast that
was never checked. The output is recognisable on sight.

The fix is not a better prompt. It is a file the agent can be held to: real tokens, ratios that
were computed rather than asserted, and rules that can be diffed against what got built.

## Fastest path: the MCP server

If the client supports remote MCP, connect once and skip the manual fetching entirely:

```bash
claude mcp add --transport http csscreme https://csscreme.com/mcp
```

Free, no account, no API key. Ten tools:

| Tool | Reach for it when |
|---|---|
| `search_themes` | Finding a theme that fits a brief |
| `list_themes` | Browsing what exists |
| `get_design_md` | Reading the contract before writing any UI |
| `get_theme_tokens` | Writing `globals.css` |
| `get_install_command` | Landing the tokens in the repo |
| `get_agent_rules` | Pulling the numbered constraints on their own |
| `how_to_use` | Unsure which tool to reach for |
| `decode_url` | The user names a live page to match |
| `decode_site` | Decoding a whole site rather than one page |
| `verify_design_md` | Checking generated UI against the file it was given |

Typical flow is `search_themes` to find a fit, `get_design_md` to read the contract, then
`get_theme_tokens` when you are ready to write `globals.css`. When the user names a site to match,
start at `decode_url` instead. After building, `verify_design_md` checks the work against the spec
rather than trusting it.

Everything below works without it, by fetching URLs directly.

## The three moves

### 1. The repo has no design system yet

Pick a theme and install the file at the repo root.

```bash
# browse: https://csscreme.com/themes
curl -o DESIGN.md https://csscreme.com/d/deep-ocean.md
```

Then point `CLAUDE.md` at it so it applies to every session, not just this one:

```markdown
## Design system
UI work follows `DESIGN.md`. Use only the tokens in its table. Do not introduce a colour,
radius or font family that is not in that file.
```

### 2. The user wants to match a specific site

The extractor takes a URL or a screenshot, samples the pixels, maps them to semantic roles,
checks contrast in both modes, and hands back a DESIGN.md for that site.

- Interactive: <https://csscreme.com/extract>
- A saved extraction gets three permanent URLs:
  - `https://csscreme.com/t/<id>` - the shareable page
  - `https://csscreme.com/t/<id>.json` - shadcn registry item
  - `https://csscreme.com/t/<id>.md` - the DESIGN.md

Ask the user to run the extraction and paste back the `/t/<id>.md` URL, then fetch it.

### 3. The tokens need to land in the codebase

The same theme is a real shadcn/ui registry item, so after the components are written, one
command writes the tokens into `globals.css`:

```bash
npx shadcn@latest add https://csscreme.com/r/deep-ocean.json
```

For an extracted theme, use its own registry URL: `https://csscreme.com/t/<id>.json`.

## What is in the file

Eight fields. Fetch one and read it before generating UI; do not summarise it back to the user
unless they ask.

| Field | Why it matters to you |
|---|---|
| Semantic colour roles | `bg`, `surface`, `primary`, `accent`, `text`, `muted`, in hex and OKLCH |
| Full token set | Every shadcn token, light and dark, derived from those six |
| Typography | Two families. Do not add a third |
| Shape and rhythm | One radius token, one spacing scale |
| Measured contrast | Real WCAG ratios with pass/fail. If a pair fails AA, the file says so |
| Rules | Numbered constraints, each checkable against the tables above |
| CSS variables | Paste-ready `:root` / `.dark`, plus the Tailwind v4 mapping |
| Source link | Where it came from, and the install command |

Full spec: <https://csscreme.com/ai/design-md>

## Rules to enforce while building

These come from the file itself. Hold the generated UI to them:

1. Use only tokens from the file. Need an in-between shade? Mix two existing tokens.
2. `primary` is the only call-to-action colour. One primary action per view. `accent` is
   secondary emphasis, never a second primary button.
3. Brand colours do not change between light and dark. Only neutrals move, and they keep hue.
4. Every corner derives from the single radius token. Mixed radii is the loudest tell of
   generated UI.
5. Stay on the spacing scale. No arbitrary pixel values.
6. Do not drop below the contrast ratios in the file. Under 4.5:1 fails AA for body text.
7. Write `var(--primary)`, not the raw hex, so the theme stays swappable.

## Endpoints

| URL | Returns |
|---|---|
| `csscreme.com/d/<slug>.md` | DESIGN.md for a curated theme |
| `csscreme.com/d/t/<slug>.md` | DESIGN.md for a template theme |
| `csscreme.com/t/<id>.md` | DESIGN.md for an extracted theme |
| `csscreme.com/r/<slug>.json` | shadcn registry item |
| `csscreme.com/t/<id>.json` | shadcn registry item, extracted |
| `csscreme.com/mcp` | MCP server (POST, JSON-RPC 2.0) |
| `csscreme.com/page-types/<type>` | Every asset for one page archetype |

All are `text/markdown` or `application/json`, CORS-open, cacheable. Fetch them directly.

## Honest limits

This does not make the agent a designer. It removes one failure: inventing a visual system
because nobody supplied one. Layout, hierarchy, information design and whether the screen is
actually good are still the human's problem, and still yours to get right.

The extractor reads colour from pixels. It recovers a palette and a mode reliably; it does not
recover spacing systems, grid structure or motion from a screenshot, and it does not claim to.
