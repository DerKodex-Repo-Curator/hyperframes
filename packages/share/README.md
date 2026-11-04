# @hyperframes/share

Node-only sharing kernel for Desktop, the CLI and local Studio's server. All hosts use the same review project, link settings, video versions and persisted retry state. This package does not render videos, sign users in, or define plan policy.

```js
import {
  bindShareHost,
  reviewClient,
  signedInRequest,
  projectFingerprint,
  shareForReview,
  addVersion,
  reviewStatus,
  reviewVersion,
} from "@hyperframes/share";

bindShareHost({
  normalize: realpathSync,
  channelSite: "https://hyperframes.dev",
  signinSites: ["https://hyperframes.dev"],
  signedInRequest,
  projectFingerprint,
  latestExportFile, // (dir) => { path, fingerprint, id } | null
  ffBinary, // ('ffprobe') => executable path
});
const client = reviewClient({ access, signedInEmail });
const share = await shareForReview({ projectPath, title, client, audience: "public" });
await addVersion({ projectPath, title, client, onProgress });
const [current, version] = await Promise.all([
  reviewStatus({ projectPath, client }),
  reviewVersion(projectPath),
]);
```

`access(refusedToken?)` returns `{ token, heygen, heygenApi, owner }` or null. It is read for every request. A refused bearer is passed back so the host can refresh it. `heygen` is the review API base. A host can inject its own `signedInRequest` to retain network tracing. `execFile` can likewise be injected to retain native process policy.

Before first registration the host must obtain explicit audience consent and prepare a current MP4. `shareForReview` registers/enables the link; it does not imply playable media. Await `addVersion`, then check the server project/latest ready version before reporting success. `addFirstVersion` starts background work and is for long-lived hosts only. `open: () => boolean` on `addVersion` bounds the polling lifetime without discarding the saved operation. `filesForExport(projectPath, renderedFingerprint)` optionally supplies an already-synced matching snapshot. Omit it for ordinary video sharing.

`reviewClient` exposes workspace capabilities, project listing/removal, grants, revision-checked access changes and export operations. `setReviewAccess`, `turnOffReviewLink`, and `removeReviewProject` retain the Desktop behavior. Enabling a revoked link produces a new URL. Disabling keeps the project and comments; removal is a separate operation.

Identity remains in the project’s generated review.json metadata (or the existing backend-specific filename); pending export requests remain in the generated review-export.json metadata. No bearer is stored there. A copied folder must deliberately reset sharing identity when creating a new project; a renamed folder retains it. A malformed/unreadable record fails instead of silently registering a new project.

Metadata mutations and version uploads use separate cross-process locks under `.hyperframes`. Locks are acquired atomically with complete owner metadata. A lock is reclaimed only when the same machine proves its owner process is dead; live, foreign-machine and malformed locks are not stolen. Interrupted upload operations resume with their saved idempotency keys. Concurrent identical uploads re-read the completed version under the lock and do not create another version.

`projectFingerprint`, `snapshotProject`, and `snapshotFingerprint` use the same HTML-stamp-insensitive source comparison as Desktop. Export/history metadata, caches, hidden folders and installed packages are excluded. Hosts must record the fingerprint of the actual rendered source and must not relabel an old render with a newer fingerprint.

Public type exports include `ShareHost`, `ReviewClient`, `ReviewShare`, `VersionRequest`, and `Wire`, plus `@hyperframes/share/types/reviewWire` and `types/share`. Per-module entry points preserve Desktop imports during adoption.

## Source snapshots

`shareSources({access, spaceId})` prepares editable source for a rendered version:

```js
const sources = shareSources({ access, spaceId });
const result = await sources.prepare(projectPath, renderedFingerprint);
if (result.at === "ready") {
  await addVersion({
    projectPath,
    title,
    client,
    filesForExport: sources.filesForExport,
    requireSource: true,
  });
}
```

Handle `unavailable.reason` before creating the video version. Plan ineligibility
can use video-only; changed source, conflicts and unexpected upload failures need
an explicit choice to omit source. The CLI exposes `--video-only` for that choice.
`filesForExport` only validates already-prepared source; it does not upload.

The adapter extracts Desktop's portable-source-v4 capture/manifest rules and
multipart transport. The pinned policy, Unicode 15.1 registry (with its license),
and server manifest vectors travel with the package. Source is captured through
verified file handles; upload bytes are streamed and checked against their hash.
Workspace limits, upload receipts and published snapshot identity are checked.
Pending uploads retain their original idempotency key across retries. A conflict
or changed pending source is never auto-resolved or cancelled.

Default state uses `.hyperframes/share-sync.<backend-hash>.json`, with strict
account/backend/realpath binding, private atomic writes, and a process lock.
Hosts can inject a compatible `store` or read-only `existingBinding` resolver.
An external Desktop binding is reused only when exact and settled; this adapter
never writes Desktop's binding or starts a second upload on its behalf.

Deployment prerequisite: review OAuth and v3 Composition Sync must admit the
CLI OAuth client, and Begin must accept `client_type: "cli"` (backend draft
[EF #64537](https://github.com/heygen-com/experiment-framework/pull/64537)).
Source-plan/feature eligibility remains server controlled. Automated fixtures do
not establish live entitlement, moderation, source download or outside-editor
acceptance. See [the source guide](../../docs/developers/share-source.mdx).
