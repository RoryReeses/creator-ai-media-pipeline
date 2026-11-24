# Reversible AI Media Pipeline for Creators


> **Decision:** Use this pipeline when media work needs rights checks, frozen inputs, human editorial decisions, reproducible rendering, and reversible release evidence; skip it for one-click clipping that cannot preserve custody or withdrawal paths.
>
> **For:** creators, media-tool maintainers, and editorial operations teams  ·  **Not for:** rights-unclear inputs or fully autonomous publishing workflows

A useful media workflow is not “paste a link and trust the clip.” It is a custody chain: establish the right to process an input, freeze its technical identity, derive transcripts and scene candidates, let a human make editorial decisions, render without losing timing, and retain enough evidence to revise or withdraw the result.

This guide helps creators and maintainers design that chain with open components. It publishes no benchmark winner and makes no claim that the projects below interoperate without adapters. The included [`pipeline-contract.yaml`](assets/pipeline-contract.yaml) is the main deliverable: it records the handoff and acceptance contract for every stage.

## Begin with delivery acceptance

Write the delivery contract before choosing a model or editor:

- destination aspect ratio, frame rate, resolution, codec, loudness, and caption format;
- whether the result must remain editable as a timeline, not just a flattened video;
- transcript language, speaker-label policy, and acceptable timing error;
- maximum clip duration and the editorial reason for each cut;
- required credit, source reference, consent, and provenance disclosure;
- who may approve a quote, crop, synthetic element, voice, or publication;
- how to correct or withdraw a delivered asset.

If the only retained object is an exported MP4, the workflow is not reversible. Keep the immutable source hash, probe output, timebase, transcript revision, edit-decision data, render command/project, and review record.

## Follow one asset through custody

### 1. Intake and rights

Accept only media that the operator owns or is authorized to transform. Record source, license or permission, people whose voice or likeness appears, territorial or platform constraints, and an expiry or withdrawal path. A public URL is not permission. Download capability is not a rights determination. The [US Copyright Office digital-replicas report](https://www.copyright.gov/ai/Copyright-and-Artificial-Intelligence-Part-1-Digital-Replicas-Report.pdf) analyzes policy questions around unauthorized replicas of voice and appearance, while the [FTC voice-cloning challenge notice](https://www.ftc.gov/system/files/ftc_gov/pdf/Voice-Cloning-Challenge.pdf) documents concrete consumer-harm concerns. Both are US sources, not worldwide clearance rules: obtain jurisdiction- and platform-specific advice for material rights questions.

Use [MediaInfo](https://github.com/MediaArea/MediaInfo) or `ffprobe` from [FFmpeg](https://ffmpeg.org/ffmpeg-filters.html) to freeze technical identity. Preserve the original; normalize into a derived mezzanine rather than overwriting it. Record variable-frame-rate behavior and rotation metadata before trusting timestamps.

### 2. Speech and alignment

[Whisper](https://github.com/openai/whisper) supplies a speech-recognition reference. [WhisperX](https://github.com/m-bain/whisperX) documents timestamp alignment and optional diarization paths. Their presence does not prove accurate words, speakers, languages, or timecodes for a particular recording.

Store raw model output separately from corrected transcript text. A reviewer should confirm names, numbers, negation, quotations, and speaker attribution against the source. Never create a quote by joining distant phrases without disclosing the edit.

### 3. Visual segmentation

[PySceneDetect](https://www.scenedetect.com/docs/latest/) can propose scene boundaries. Audio silence, slide changes, speaker changes, and semantic topic changes are different signals; none is automatically an editorial cut. Candidate scoring should retain the reason and source interval so a human can reject it.

The author recommendation in this guide is to require at least two independent cues for automatic **suggestion**, while reserving selection to a reviewer. That threshold is a workflow policy, not a claim made by PySceneDetect.

### 4. Edit decisions

[auto-editor](https://github.com/WyattBlue/auto-editor) can automate timeline edits; [LosslessCut](https://github.com/mifi/lossless-cut) supports rapid lossless-oriented cutting; [Remotion](https://www.remotion.dev/docs/) offers programmatic composition. These are different roles, not substitutes on a speed leaderboard.

Represent an edit as source intervals, transforms, overlays, and reasons. Keep handles around cuts. Treat reframing, face tracking, denoise, translation, and synthetic B-roll as separately approved transformations. A human must inspect whether a crop removes evidence or changes meaning.

### 5. Captions and render

[WebVTT](https://www.w3.org/TR/webvtt1/) defines a web caption format; it does not specify the quality of generated text. Preserve a master caption track independently from burned-in captions. Validate monotonic cue times, overlaps, reading speed, line breaks, safe areas, and text equivalence after render.

Use FFmpeg filters only with a frozen command and version. Decode several points near each cut, compare expected duration, and inspect audio sync. Hardware encoders, color conversion, fonts, and platform transcodes can change output even when the edit list is unchanged.

### 6. Provenance and release

The [C2PA 2.2 specification](https://spec.c2pa.org/specifications/specifications/2.2/specs/C2PA_Specification.html) provides a provenance framework. A manifest can describe history and assertions; it does not prove truth, ownership, consent, or editorial fairness. Keep the human rights ledger and review evidence even when C2PA is used.

Release only after the approver can answer: what source intervals were used, what was corrected or synthesized, whose approval is recorded, and how can this version be withdrawn? Publication remains a separate action outside this package.

## Stage, component, and handoff map

This matrix is a role map, not an interoperability claim. Adapters, file conventions, versions, and licenses still require local confirmation.

| Stage | Candidate role | Receives | Emits | Useful alternative or complement | Handoff that must be verified |
|---|---|---|---|---|---|
| Inspect | MediaInfo or FFmpeg probe | immutable source bytes | stream, codec, duration, rotation, and timebase metadata | use both when container and decoder views differ | source hash and timebase remain attached to every derivative |
| Transcribe | Whisper | normalized or original audio | segments and text | another authorized STT runtime may replace it | language, model identity, raw output, and corrected revision are separate |
| Align/diarize | WhisperX | audio plus transcript | refined word timing and optional speaker labels | manual checkpoints remain necessary | timestamps map to the frozen source, not an overwritten transcode |
| Segment | PySceneDetect | video frames/timebase | candidate scene boundaries | audio, slide, or semantic cues complement visual cuts | candidates retain source intervals and reasons; they are not edits |
| Edit | auto-editor or LosslessCut | source intervals and edit policy | automated timeline or cut list | LosslessCut favors rapid cut review; auto-editor expresses automated rules | exported decisions remain reversible and source-addressed |
| Compose | Remotion | approved media, captions, overlays, timing | programmatic composition and render inputs | a conventional editor can replace this role | fonts, assets, code version, and render configuration are frozen |
| Caption | WebVTT | corrected text and approved timing | portable caption cues | platform-native formats may be derived | cue timing and text equivalence survive conversion and burn-in |
| Render/QC | FFmpeg | approved timeline or composition | delivery file and logs | hardware or application renderer may replace it | duration, A/V sync, color, captions, and checksums meet the contract |
| Provenance | C2PA plus the human rights ledger | identities, transformations, approvals | signed assertions where supported and a withdrawal record | neither replaces the other | provenance assertions never stand in for ownership or consent |

No row means the named projects connect directly. Prove each adjacent pair with an authorized fixture before adopting the route.

## Quality gates that fail closed

| Gate | Evidence | Reject or hold when |
|---|---|---|
| Intake | source hash, rights basis, probe record | authority is unknown, source changed, or technical metadata is unresolved |
| Transcript | raw output, corrected revision, reviewer | key claims/names are unverified or speaker attribution is uncertain |
| Selection | source intervals and editorial rationale | context changes meaning or a quote is assembled deceptively |
| Transform | parameter/config record | crop, translation, synthetic media, or voice use lacks approval |
| Render | command/project, duration and sync checks | timebase, caption, color, audio, or codec acceptance fails |
| Release | rights ledger, provenance, approver, withdrawal path | consent, attribution, disclosure, or correction ownership is absent |

A model confidence score must not waive a gate. “Local” execution does not establish privacy if temporary files, model telemetry, plugins, or review uploads leave the controlled system.

## A small comparison experiment

To compare two candidate pipelines, use the same authorized source, source intervals, transcript corrections, delivery contract, and hardware declaration. Measure:

1. transcript word and key-entity corrections after human review;
2. alignment deviations at predeclared checkpoints;
3. edit decisions accepted/rejected and the reviewer reason;
4. end-to-end operator time, not only model runtime;
5. rendered A/V sync, caption validity, and output size;
6. reversibility: time required to change one cut and regenerate;
7. artifacts retained and data sent outside the controlled boundary.

Do not combine a different model, source, codec, and editor into one claimed comparison. A convenient pipeline may be preferable without being “more accurate.”

## Rights, identity, and harm boundary

- Do not process media without a valid permission or legal basis.
- Do not imitate a person’s voice, likeness, or endorsement without explicit authorization.
- Do not strip attribution, watermark, provenance, or disclosure to mislead viewers.
- Do not present generated or translated words as a verbatim quote without review.
- Do not use this workflow to evade platform rules, copyright controls, privacy obligations, or takedown requests.
- For minors, medical/legal testimony, intimate media, or vulnerable people, require a stricter human/legal review outside this guide.

## What the linked sources support

The ten source records in [`sources/sources.json`](sources/sources.json) support narrow format, project, or feature roles. They do not collectively endorse this pipeline. Stage order, dual-cue suggestion, gate policy, the custody model, and the comparison experiment are author recommendations that require adaptation to jurisdiction and production context.

## Maintenance

Re-run acceptance after changing source, model, alignment method, editor, fonts, renderer, hardware encoder, platform target, or rights status. Preserve old manifests and rendered checksums. Never overwrite a failed run with a successful rerun; link them so the correction remains visible.

## Adjacent guides

- [ai-research-reproducibility](https://github.com/Lindenberg668/ai-research-reproducibility) — Use it when a media-derived claim also needs immutable inputs, procedure, environment, and variance records.
- [open-source-ai-code-provenance](https://github.com/ankr989/open-source-ai-code-provenance) — Use it when software artifacts in the pipeline need a narrower AI-assistance provenance envelope.
- [self-hosted-notification-routing-index](https://github.com/brownrasr/self-hosted-notification-routing-index) — Use it when review, render, or withdrawal events must reach people through a controlled notification path.

## License

Original guide and pipeline contract: CC BY 4.0. Linked projects and specifications retain their own terms.

If this saved you a decision, star the repo — it is how the next reader finds it.
