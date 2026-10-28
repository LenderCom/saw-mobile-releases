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
- **Yank, never delete.** To pull a build, re-publish its `manifest.json` with `"yanked": true`, signed with the
  same channel key. agents-api then stops offering it. Deleting a head release would let an older build back in
  as the head.
