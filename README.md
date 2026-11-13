# Nixie.Studio plugin for Claude Code

Make music videos and narrated explainer or training videos with
[Nixie.Studio](https://nixie.studio) from a Claude Code session.

## What it does

The plugin adds two things:

- **The Nixie MCP connector** (`.mcp.json`): Nixie's remote MCP server at
  `https://nixie.studio/mcp`. It gives Claude the Video Brief tools:
  `discover_video_contract`, `submit_video_brief`, `run_video`, `check_status`,
  `get_result`, `attach_character`, `attach_song`, `plan_broll`, `regenerate_shot`,
  `recut_video`, `halt_run` and the rest.
- **The `nixie-video` skill** (`skills/nixie-video/SKILL.md`): the method. Read the
  contract first, submit a brief, read the estimate and the warnings, run with a
  spending ceiling, poll the status to the end, and read the result. It also covers
  characters, songs, b-roll and what to do when a scene parks or a run warns.
  Run it yourself with `/nixie-studio:nixie-video`, or let Claude load it when you ask
  for a video.

## Install

From a Claude Code session:

```
/plugin install nixie-studio@nixie-studio
```

That works once the `nixie-studio` marketplace is added. To add it and install in one
step:

```
/plugin install nixie-studio --marketplace taburley/nixie-studio-plugin
```

Or from your shell:

```
claude plugin marketplace add taburley/nixie-studio-plugin
claude plugin install nixie-studio@nixie-studio
```

To try it for one session without installing: `claude --plugin-dir <path to this folder>`.

## Sign in

Until you sign in, Claude Code lists the plugin's `nixie` server as needing
authentication. Run `/mcp`, pick the Nixie server and sign in in your browser. Nixie shows a consent
screen naming the Nixie account you are approving; the connection then runs as that
account. You need a Nixie.Studio account.

Already connected to Nixie another way (a claude.ai connector, or `claude mcp add`
with an API key)? Then you would see two sets of the same tools. Keep one: turn the
plugin's server off in `/mcp`, or remove the other connection.

**Headless use (scripts, CI).** The plugin itself only uses OAuth sign-in. For a run
with no browser, add the server yourself with a Nixie API key read from your
environment, as described on
[Nixie's MCP help](https://nixie.studio/help#api-access/mcp-clients-a-key-or-a-consent-screen),
and turn the plugin's server off in `/mcp`. Never paste a key into this plugin's files.

## Cost

The plugin is free and spends nothing on its own. Nixie renders on your own AI
provider keys. Every tool that spends takes a spending ceiling (`ceilingUSD`), and the
skill tells Claude to state the estimate before any paid step. Current prices and caps
come from Nixie itself (`discover_video_contract`, `list_models`), not from this
plugin.

## Privacy and data

The plugin contains no data, no keys and no secrets: only a server address, a skill
and this README. When you use it, the scripts, prompts, pictures, songs and other
inputs you ask Claude to pass to the Nixie tools are sent to nixie.studio under your
account, and the results (stills, clips, narration, the finished video) are stored in
your Nixie account.

## Help

- Studio help: https://nixie.studio/help
- Problems with a run: ask Claude to file it with the `report_issue` tool, which sends
  it to Nixie with the details of what was called.

## Licence

MIT, see [LICENSE](LICENSE). The licence covers the files in this repository (the
plugin manifest, the connector address and the skill). Nixie.Studio itself, the service
these files connect to, is not part of it.
