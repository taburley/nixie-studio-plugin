---
name: nixie-video
description: Make, plan, render, re-roll, recut or check a video on Nixie.Studio through its MCP tools - a music video from a song, or a narrated explainer or training video from a script. Use when the user wants a video made on Nixie, wants a song or a recurring character in one, wants b-roll, or asks why a Nixie run stopped, parked or warned.
---

# Making a video with Nixie.Studio

The tools come from this plugin's `nixie` MCP server (https://nixie.studio/mcp). Every call runs as the Nixie member who signed in.

## 1. Read the contract first (spends nothing)

- Call `discover_video_contract` with `section: "summary"` first, then only the sections you need (for example `request`, or `example,responses`). The whole document is too long for most clients.
- Its answer names the account this connection runs as (`connectedAs`, `note`). If that is not the user, stop and tell them.
- Prices, caps, defaults and model choices (`imageTiers`, `castModels`, `motionTiers`, `productionTypes`) live in the contract and in `list_models`. Read them there each time. Do not quote a price or a cap from memory or from this file.

## 2. The order of a run

1. `submit_video_brief` with `version: "0.1"` and `scenes`. Each scene needs `narration` and `durationSec`, and exactly ONE of `visualPrompt`, `imageUrl` or `assetId`. This only plans; it spends nothing. Avoid depicting technical equipment in a `visualPrompt`.
2. Read the answer before going on: `estimatedSpendUSD` (by line; a part the studio cannot price is null with the reason), `promptCheck` and `consistency` warnings (fix those scenes and re-submit), and `resolved`.
3. Optional, after submit and before the run: `attach_character`, `attach_song`, `plan_broll`.
4. Tell the member the estimate and agree a ceiling. Then `run_video` with `productionId` and `ceilingUSD`. It spends the member's own AI keys up to that ceiling.
5. Poll `check_status` with `verbose: false`. On every poll tell the member `progress.label`, wait `progress.nextPollSec`, and keep polling until `phase` is `done` or `failed`. Never declare the run finished yourself.
6. `get_result` for the video, share links, spend and shots. Use `get_captions` for captions; `captionsUrl` is a key-gated link.

## 3. Cost rules

- Say what a paid step will cost (its estimate) before you call it. `submit_video_brief` gives that estimate before anything runs.
- Every paid tool takes a `ceilingUSD`: `run_video`, `regenerate_shot`, `generate_asset`, `narrate`, `transcribe`, and `attach_character` when it uses `imagePrompt`. `regenerate_shot`, `generate_asset`, `narrate` and `transcribe` refuse (402 over_budget) before any provider call when the estimate exceeds the ceiling.
- A video `render.motionTier` needs its own `render.motionCeilingUSD`; the answer reports the summed cap as `capTotalUSD`.
- Paid work runs on the member's own keys. `generate_asset` and `narrate` refuse a member with no key of their own (`no_key`).
- `plan_broll` and `enhance_prompt` make one small Anthropic call; for `plan_broll` the key may be the member's, their team leader's or the studio's (`keySource` says which).
- `run_video` with `force: true` re-runs a delivered production and spends again. Pass the same `idempotencyKey` to replay a run instead of starting a new one.
- If a run is spending in a way the member did not intend, call `halt_run`.

## 4. Cast: the same character in every scene

1. After submit and before the run, `attach_character` with `productionId`, `name` and exactly one of: `assetId` (an image in the member's catalog, $0), `imageUrl` (a direct link, $0) or `imagePrompt` (renders the reference on the member's key, capped by `ceilingUSD`).
2. A picture the user holds: `upload_image` (a small data URL plus a `caption`) and pass the returned `assetId`. A real screenshot is too big for a tool argument; ask the user for a direct link or to drop the file on a storyboard in the studio.
3. Put `@<slug>` (the `slug` the call returns) in each scene's `visualPrompt`. Never put an `@slug` inside `attach_character`'s own `imagePrompt`. An `@token` that matches no character is plain text and the render still spends.
4. Cast scenes render on `image.castModel`. Pick one from the contract's `castModels` or ask the member; if you leave it out the studio picks one and echoes it as `resolved.image.castModel`.
5. A 409 `name_slug_taken` or `name_taken` means choose another name and call again.

## 5. The member's own pictures and motion

- A scene can use the member's own picture as its still: `imageUrl` or `assetId` instead of `visualPrompt`. Nothing is generated or judged for that scene.
- With `render.motionTier` set to a video tier, the studio turns each kept still (generated or the member's own) into a motion clip. Describe the motion in that scene's `motionPrompt`: preserve the frame, ONE action, one named camera move with a magnitude and an end state, lighting constant. Without a `motionPrompt` the still is animated as a static hold. `motion: false` keeps a scene as a still.
- Keep each spoken line under the tier's `maxClipSec` (about `maxClipSec` x 15 characters) or the scene holds its last frame; the `clip_shorter_than_scene` warning names those scenes.

## 6. Song and b-roll

- Song: `attach_song` after submit and before the run, either with `audioUrl` (a direct link to the file, not a share page) or with no `audioUrl` to adopt the song the member uploaded on the production page. Set the scenes' `durationSec` to sum to the song length. On `no_song`, ask the member to upload it there.
- B-roll: add a `broll` block to the brief, or call `plan_broll` for a proposal and approve it by passing its `planId` as `run_video`'s `brollPlanId`. Read the `brollPlan` table on the submit answer and on `check_status`.

## 7. When something goes wrong

- **Parked or missing scenes.** A shot that casts an `@slug` whose reference cannot be resolved parks rather than render a stranger. Read `droppedScenes` (each with its reason) and `qualityCheck` on `check_status` before calling a run done. Fix the cause, then use `run_video` (with `force: true` once a cut was delivered): `regenerate_shot` refuses while another scene is still missing a take.
- **A `warning` when the phase is done.** Any non-null `warning` is for the member: tell them the sentence as written. "delivered with warnings" means part of the work was lost, and the sentence says what fixes it. `recut_video` re-attempts lost b-roll for free.
- **One bad scene in a delivered video.** `regenerate_shot` (spends, needs `ceilingUSD`), then poll `check_status` until `regenerate.status` is `done` or `failed`; the phase reads `done` either way. A 409 `scene_not_narrated` means use `run_video`.
- **Timeline-only changes** (crossfade, fps, captions, resolution): `recut_video`, which spends nothing. Generation settings need a re-submitted brief and `run_video` with `force: true`.
- **A short production id that matches several productions**: pass the full id.
- **Anything wrong or unexpected**: `report_issue` with what you called, what came back and what you expected. Put several findings in one report.

## Help

- Studio help: https://nixie.studio/help
- Connecting an MCP client: https://nixie.studio/help#api-access/mcp-clients-a-key-or-a-consent-screen
