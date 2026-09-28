# Translation Bot (ES) — Routine spec

Produces a Spanish-language sibling video for an already-uploaded English video, reusing the
original visuals to keep cost near-zero. Paste this into a new Claude Code Routine.

## Trigger

- Card moved to **"Translation Requests"** on a channel board (Latitude Zero, Threshold, ...).
- Only cards Bas explicitly drags there get processed — never auto-picks up everything in
  "uploaded". This is the cost control: ElevenLabs + render cost per language, so it's opt-in
  per video.
- On finish, move the card to **"Translation Done"** and post a comment with the Drive link
  (Trello connector can't upload attachments directly — same pattern as the Script Writer Bot).

## Required setup (one-time, per channel)

1. A Spanish sibling YouTube channel (e.g. "Latitude Zero ES", "Threshold ES") — mirrors how
   the reference channels themselves ship language siblings (Extinct World / Mundo Extinto).
2. One fixed ElevenLabs Spanish voice per channel — new `ELEVENLABS_VOICE_ID_ES` env var,
   separate from the channel's English `ELEVENLABS_VOICE_ID`.
3. A Drive output folder per channel: `Niches/<Channel>/<VideoName>-ES/`.
4. Trello card for the source video must carry (in its description or a comment, per the
   existing custom-field-unreadable workaround) the Drive links to: final script (with chapter
   markers), title, description, and the rendered video project — i.e. whatever the Video
   Generator Bot already leaves behind for that card.

## Pipeline

1. **Fetch source**: read the Trello card in Translation Requests, pull the English script,
   title, description and the original render (visuals track) from Drive.
2. **Translate** script, title, description to Spanish (not literal — culturally natural,
   same factual claims, same hedging/sourcing). Keep the existing pacing rule: first 30s must
   fully answer the title's question, next 3–5 min stays tight.
3. **Regenerate VO** with ElevenLabs using `ELEVENLABS_VOICE_ID_ES`. Spanish audio typically
   runs ~15–20% longer than English for the same content — budget for this in step 4 rather
   than cutting the translation to fit the original runtime.
4. **Re-sync visuals**: reuse the original AI stills/motion clips as-is (this is the actual
   cost saving — no new Replicate/GenAIPro generation). Retime by proportionally stretching
   each clip's hold/push-in duration to the new VO length via ffmpeg, rather than regenerating
   or reordering shots. Flag any single clip that would need >30% stretch for manual review
   instead of forcing it.
5. **Thumbnails**: reuse the existing thumbnail composition; swap only the text layer
   (title word / data label) via code-compositing in Spanish — same approach already used for
   Threshold's thumbnail text, for guaranteed correct spelling. No new base image generation.
6. **Deliver**: upload final video + 3 thumbnail variants + description to
   `Niches/<Channel>/<VideoName>-ES/` on Drive, log cost, send a Telegram notification.
   Actual YouTube upload to the ES channel stays manual (same as the main Video Generator Bot
   — it hands off to Drive, Bas/partner does the publish step).
7. Move the Trello card to **Translation Done**, comment with the Drive link.

## Hard limits

- Spanish only for now — do not add other languages without a new voice ID and explicit request.
- Never regenerate visuals from scratch — retiming only. If retiming can't preserve sync
  within tolerance, stop and flag the card rather than silently degrading quality.
- Never touches a card outside "Translation Requests" — no scanning "uploaded" directly.
- Same factual-grounding bar as the source script: translation must not introduce claims the
  English version didn't make.

## Cost estimate

Per video: ElevenLabs (~30 min VO), ffmpeg render (compute only, free), no new
Replicate/GenAIPro spend. Materially cheaper than a fresh video — no new script research, no
new visuals.
