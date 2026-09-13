# csscreme skill

A Claude Code skill that gives a coding agent a real design system before it writes UI:
semantic tokens, measured WCAG contrast, and rules it can be held to.

Free. No account, no API key, no paid tier.

## Why

An agent asked for UI with no design system does the same thing every time: invents a palette,
mixes three or four corner radii, adds a second font nobody asked for, and ships contrast that
was never checked. The result is recognisable on sight, and no amount of prompt rewording fixes
it, because the problem is missing input rather than weak instruction.

A file fixes it. Tokens the agent must use, ratios that were computed rather than asserted, and
numbered rules that can be diffed against what actually got built. "Use only these six roles" is
checkable. "Make it clean" is not.

## Install

**Fastest: connect the MCP server** (free, no account, no key):

```bash
claude mcp add --transport http csscreme https://csscreme.com/mcp
```

That gives the agent ten tools with no files to manage: `search_themes`, `list_themes`,
`get_design_md`, `get_theme_tokens`, `get_install_command`, `get_agent_rules`, `how_to_use`,
`decode_url`, `decode_site` and `verify_design_md`. The last three are the ones worth knowing
about: point the agent at a live URL and it decodes that site into the same format, then checks
its own generated UI against the file afterwards. The skill below adds the house rules on top.

### As a skill

**Claude Code (project):**

```bash
mkdir -p .claude/skills/csscreme
curl -o .claude/skills/csscreme/SKILL.md \
  https://raw.githubusercontent.com/damnepic/csscreme-skill/main/SKILL.md
```

**Claude Code (all projects):** same, into `~/.claude/skills/csscreme/`.

Then ask for UI as normal. The skill fires on things like "build a dashboard", "make this look
less generic", or "match the design of stripe.com".

## What it does

1. **Fetches a DESIGN.md** for one of the curated themes at <https://csscreme.com/themes>.
2. **Or extracts one from any site** via <https://csscreme.com/extract> - URL or screenshot in,
   role-mapped tokens out, contrast checked in both modes.
3. **Installs the tokens for real** with `npx shadcn@latest add <registry-url>`.

## Endpoints it uses

| URL | Returns |
|---|---|
| `csscreme.com/d/<slug>.md` | DESIGN.md for a curated theme |
| `csscreme.com/d/t/<slug>.md` | DESIGN.md for a template theme |
| `csscreme.com/t/<id>.md` | DESIGN.md for an extracted theme |
| `csscreme.com/r/<slug>.json` | shadcn/ui registry item |
| `csscreme.com/t/<id>.json` | shadcn/ui registry item, extracted |
| `csscreme.com/mcp` | MCP server (JSON-RPC 2.0 over POST) |

All CORS-open and cacheable, served as `text/markdown` or `application/json`.

## What the agent actually gets

A DESIGN.md is a contract rather than a description. Eight fields: semantic colour roles in hex and
OKLCH, the full shadcn token set in light and dark, two font families, one radius token, an explicit
spacing scale, WCAG contrast ratios that were computed rather than asserted, numbered rules each
checkable against the tables above them, and paste-ready CSS variables.

The contrast section is the part worth reading twice. If a colour pair fails AA, the file says
`FAILS WCAG` instead of quietly shipping it. Nothing in the file is an adjective the agent has to
interpret.

## Works with other tools too

The files are plain Markdown, so nothing here is Claude-specific. Per-tool wiring for Cursor,
Codex, v0 and Lovable: <https://csscreme.com/ai/design-md>

## The format

Full field spec, with a worked example generated at build time:
<https://csscreme.com/ai/design-md>

Nobody owns the `DESIGN.md` file name, us included. The useful part here is the generator and
the measured contrast, not the noun.

## Licence

MIT. The skill is MIT; the themes it fetches are free to use.
