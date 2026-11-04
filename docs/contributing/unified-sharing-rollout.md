# Unified local sharing rollout

This is the release checklist for replacing OSS publishing with the same review links used by Desktop. These are draft changes, not deployed behavior. Five active free shares remain server-enforced; turning a link off does not release its slot, while removing a share does. The legacy viewer remains reachable through the web app.

## Candidate ledger

Record the final approved heads again after stack rebases. The following are the implementation checkpoints, not release approvals:

| Slice | Draft | Checkpoint |
| --- | --- | --- |
| Backend CLI authentication and five-free sharing | experiment-framework #64537 | `ed5d73322215c94cf2c618862ab7ea8d6907072a` |
| Backend Composition Sync CLI client | experiment-framework #64548 | `9cd3481687fd871aa30b3f40b1850eb2e484cc4c` |
| Public share kernel | hyperframes #7134 | `9944fa97c76d02c9493e1e08d5358650efceac6b` |
| CLI share | hyperframes CLI stack | `7d4a332ba` |
| Source adapter | hyperframes source stack | `b15a9d67c` |
| Desktop adoption | hyperframes-internal #8386 | `9d13fc0973` |
| Legacy archive entry | hyperframes-internal #8382 | `0c97e4cfb0e66e1dc5c61f8c7fde5eea8c84cd5d` |
| Publish compatibility transition | hyperframes #7136 | `086172e45e62cd9de33d0234796aec9f51251536` |
| Lifecycle analytics and local Studio | dependent stack | Record final heads before acceptance |

Before release, record the OSS npm version/integrity, Desktop version, backend deployment revisions, web deployment ID, OAuth client/environment, effective review flag responses, and quota configuration alongside each accepted head. A merged PR is not a deployed revision.

## Acceptance before enabling entry creation

- Check dedicated CLI OAuth on dev: signed out, expired session, refresh, account switch, wrong backend and API-key-only credentials. No token or secret URL may enter logs or analytics.
- Confirm the backend admits CLI and Desktop through the same review API and quota. Create five distinct active shares, refuse a sixth before an unnecessary render, update an existing share at the cap, disable a link without freeing a slot, remove one and create another. Verify workspace restrictions and explicit public consent.
- On one project folder, create in CLI, inspect in local Studio and Desktop, then update from each. Verify unchanged identity and URL, concurrency protection, lost-response retry, killed-process recovery, changed-source refusal, screening hold and timeout. A prior ready video must never be reported as a completed update.
- Verify eligible sources use the same Composition Sync association and exclusions. Free video-only sharing stays available. Explicit eligible source requests must surface a source upload failure rather than silently omit sources. Test the explicit video-only choice.
- Verify settings use owner capabilities: visibility, link off/on, removal, invitation/grant management and unavailable entitlements. The new local entry must not expose old publish controls.
- Walk signed-out, signed-in owner and reviewer URLs on the dev backend. Capture the selected Quick Share UI in both themes and narrow widths. Record the exact candidate head with the capture.
- Verify the legacy archive link and previously published URLs. Confirm the CLI transition message and preserved identity files without silently converting legacy pages.
- Run package tests, declaration/type checks, production builds, Desktop packaging-lock checks and the signed-in cross-surface walk on final heads. Build Desktop from the published registry kernel through `pnpm hf:pin`, not the draft pnpm Git subtree dependency.

## Entry rollback

The authoritative server response read by `ReviewClient.reviewFlags()` maps `version_sharing` to `versionSharing` and `share_files` to `shareFiles` (backend flags `hyperframes-review-version-sharing` and `hyperframes-review-share-files`). The shared CLI service checks `versionSharing` before rendering, first registration or version upload. The local Studio bridge uses that same service and reports the same capabilities.

Use the existing backend-owned review flag to stop new local share mutations for the affected cohort. Validate its effective response for the exact OAuth client and workspace; do not assume a UI flag controls authorization. Existing ready-link retrieval, owner management and legacy URLs remain available. Link reactivation requires an existing review, so it cannot be an alternate creation endpoint. Quota remains enforced server-side throughout. Never fall back to legacy publishing, bypass screening, or mint a second identity to recover a refusal.

A flag change cannot cancel an upload already admitted. Observe in-flight processing and reconcile its final state; preserve retry records, review rows, tombstones and media. If wider transport or playback fails, use the owning backend/release rollback procedure, preserving these durable records. This checklist does not apply any flag or deploy anything.

## Metrics and privacy

New event properties are restricted to `client_surface` (`cli` or `local_studio`), action (`share` or `update`), a random per-invocation `attempt_id`, boolean upload-recovery state and finite failure reasons. There are no project paths, titles, workspace/account IDs, emails, share URLs, tokens, raw errors or source manifests in these properties. The existing CLI telemetry transport retains its already-disclosed identity/system metadata and opt-outs. Persisted telemetry preference is refreshed for each event so an open preview honors an opt-out from another terminal.

| Event | Exact boundary |
| --- | --- |
| `share_mutation_attempted` | A share/update intent enters the service; a repeated share may retrieve an existing link. |
| `share_mutation_authenticated` | The host's OAuth access read succeeds, before owner state is fetched. |
| `share_recovery_attempted` | The operation observes a pending export record. This does not prove recovery succeeded. |
| `share_mutation_admitted` | The kernel reports uploading, processing or checking after an accepted export response; emitted at most once per attempt. A verified newly completed export also establishes admission. Local preflight alone never counts. |
| `share_video_ready` | This attempt added a version and its owner status confirms a ready active link. No event on old-ready, timeout, deduplicated upload or repeated retrieval. |
| `share_mutation_processing` | The bounded call returns without a new ready result; it is not a failure or success. |
| `share_mutation_failed` | The operation throws; only an allowlisted reason is sent. Unknown errors are `other`. |
| `share_existing_link_returned` | An existing link/state is retrieved or an identical completed upload is reused. Exclude from new-upload conversion. |
| `share_opened` | Local Studio's share dialog is opened, with Desktop-compatible `link=none/live/changed`. Not reviewer-page navigation. |
| `share_link_copied` | Clipboard write succeeds, with Desktop-compatible `via=copy/publish/update`. Printing a CLI URL never counts as copying. |

Join lifecycle stages by `attempt_id`, segment by surface and action, and deduplicate repeated event delivery. Report attempted → authenticated → admitted → ready, with existing retrieval and still-processing as separate outcomes. Report recovery attempts and ready results with `recovery=true`; do not call a later polling read a recovered upload. Count failures by stage/reason, especially quota, auth, moderation and feature unavailable. Surface-copy counts measure actual clipboard success, not reviewer reach; reviewer visits/comments remain the share-page funnel. Desktop's existing names are prefixed `desktop_`; normalize their dialog/copy semantics explicitly rather than joining every event name blindly.

Telemetry is optional and best effort: rates cover observed opted-in attempts, not all users. A killed process may leave only an attempted/admitted event. Server counts are the authority for quota and durable completed versions. Set thresholds from dev/canary baselines and record the chosen thresholds before rollout; no fabricated numeric baseline is included here.

## Legacy transition clock

Release date: **unset**. Sunset date: **unset**. The approved transition lasts 60 days from the user-facing release, not from this draft or merge. When the release is approved, record its version and date, calculate the cutoff, and update the CLI notice and support material together. Keep old published projects and the archive entry reachable after creation is retired; no automatic deletion is part of this rollout.
