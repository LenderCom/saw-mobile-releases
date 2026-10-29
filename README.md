# saw-mobile-releases

Public, signed release artifacts for **SAW for Android** (the sideloaded APK). The app itself is built in
`LenderCom/saw-mobile`; this repo only holds its releases (SAW-3526).

## Download

| Channel | Newest build (a link that never changes) | Offered by |
|---|---|---|
| `android` (stable) | [`android-latest/SAW-android.apk`](https://github.com/LenderCom/saw-mobile-releases/releases/download/android-latest/SAW-android.apk) | the prod cloud |
| `android-dev` | [`android-dev-latest/SAW-dev-android.apk`](https://github.com/LenderCom/saw-mobile-releases/releases/download/android-dev-latest/SAW-dev-android.apk) | the dev cloud |

Signed-in users also find the link, with a QR code, in SAW's **Download** dialog (account menu). Open the
downloaded file on the phone. The first time, Android asks you to allow installs from that source. After that,
the app tells you when a newer build is out and installs it itself.

## Releases

- **`dev-v1.0.<n>`**: a *prerelease* for every push to saw-mobile `main` (ci.yml `android-publish`). `n` is the
  build's versionCode.
- **`v1.0.<n>`**: a *release*, marked Latest, when a dev build is promoted to stable (saw-mobile
  `android-promote.yml`, a manual dispatch). It carries the same APK as `dev-v1.0.<n>`, not a rebuild.
- **`android-dev-latest`** / **`android-latest`**: prereleases whose APK is replaced on every publish or
  promote. They exist only for the permanent links above, and are prereleases so they never become Latest.
  Each carries a `version.txt` naming the build it serves.

Each versioned release carries `SAW-android-1.0.<n>.apk`, `manifest.json` and `manifest.json.minisig`. The
manifest uses the same schema as saw-agent and saw-desktop: version, channel, commit, min_version, and the APK's
sha256 and size. It is minisign-signed with the channel's key:

| Channel | Public key |
|---|---|
| `android-dev` | `RWTJ2TJ+VTpIlOQ7NzrtZTlkYxHu/1xOFvqQYc9eCvuVtK8tJCjrtxGG` |
| `android` | `RWRWf98OwlnQYgaMteYJr263/e54LMiZCgsXRukVIagNtlJtuljBw894` |

agents-api's manifest refresher verifies each channel's head and serves it at
`GET /api/v1/channels/{android-dev|android}/downloads`. The Download dialog and the in-app updater read that,
and the updater checks the downloaded APK against the manifest's sha256. Android also refuses any APK not signed
with the installed app's key.

## Rules

- **Publishing is CI-only.** The private signing keys (`MOBILE_MINISIGN_ANDROID_DEV_KEY`,
  `MOBILE_MINISIGN_ANDROID_KEY`) live in saw-mobile's Actions secrets.
- **Heads only move forward.** A promote refuses a version older than the current `v*` Latest. A permanent
  `*-latest` link moves only to a newer build than the one it holds, so an older CI run that finishes late
  leaves it alone. Only a yank moves a link back.
- **Yank, never delete.** To pull a build, run saw-mobile's **Yank an Android build** workflow
  (`android-yank.yml`, a manual dispatch with the channel, the version and an optional reason). It re-publishes
  the build's `manifest.json` with `"yanked": true`, signed with the same channel key, and marks the release
  **Yanked**. agents-api then stops offering it. If the channel's `*-latest` link served that build, the
  workflow moves the link back to the newest build on the channel that is not yanked, or removes the link's APK
  when none is left. Deleting a head release would let an older build back in as the head. To ship past a
  yanked build, publish or promote a newer one.
