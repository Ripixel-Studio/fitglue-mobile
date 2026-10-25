# Retiring FitGlue Mobile — the by-hand checklist

Written 2026-10-25 for task F0.2 of the single-user conversion
(`jarvis-memory/specs/fitglue/personal-fitglue.md`). FitGlue is now James's
personal project; the Apple Health and Health Connect sources are gone, so this
app is retired. The goal of this list is to make sure the app cannot be
installed, built or submitted by anyone, **without** closing the Apple or Google
developer accounts. Those stay open; what they cost and whether to renew is a
separate decision for James.

The repo side is done in the PR that added this file (CI and EAS workflows
removed, `eas.json` removed, README marked retired). After that PR merges the
repo gets archived with `gh repo archive Ripixel-Studio/fitglue-mobile -y`.
Everything below is console work only James can do.

## Identifiers

| Thing | Value | Where it came from |
|---|---|---|
| iOS bundle identifier | `com.fitglue.mobile` | `app.json` → `ios.bundleIdentifier` |
| Android package name | `com.fitglue.mobile` | `app.json` → `android.package` |
| Expo account / slug | `@ripixel/fitglue-mobile` | `app.json` → `slug`; EAS workflow actor |
| EAS project id | `30cf5f2d-a1e9-43df-864b-28aa0aa26217` | `app.json` → `extra.eas.projectId` |
| EAS Update URL / channel | `https://u.expo.dev/30cf5f2d-…` / `production` | `app.json` → `updates`, `eas.json` |
| EAS build profiles | `development`, `preview`, `production` (store), `production-apk`, `local-verification` | `eas.json` (now deleted) |
| Firebase project | `fitglue-server-prod` (GCP project number `605889586984`) | `google-services.json`, `GoogleService-Info.plist` |
| Firebase Android app id | `1:605889586984:android:8cfb019f110aa6cd50521d` | `google-services.json` |
| Firebase iOS app id | `1:605889586984:ios:9375a364e0edb13650521d` | `GoogleService-Info.plist` |
| Google OAuth client in `google-services.json` | `605889586984-k8v7qglu8kcsfkb5nqkiuemvjt3cf9d7.apps.googleusercontent.com` (type 3 = web client) | `google-services.json` |
| Sentry | org `ripixel-studio`, project `react-native` | `app.json` → `@sentry/react-native` plugin |
| Release tag convention | `mobile/v{version}` | `CLAUDE.md` |

## What the live state looks like (checked 2026-10-25 via the Expo API)

Before you go clicking, know that this app may never have reached either store:

- EAS has **no finished store build on either platform**. Every finished build is
  an Android `production-apk` internal build (the sideloaded APK James ran on
  his own phone; last one 2026-06-16, commit `33412e0`). The only two
  store-distribution builds — Android `production` on 2026-05-26 and iOS
  `production` on 2026-02-17 — both errored in the prebuild/config phase.
- There is **no iOS build that ever finished**, so nothing was ever uploaded to
  TestFlight or App Store Connect through EAS.
- The Expo account has **no App Store Connect API key and no Google Play service
  account key** attached, so `eas submit` was never configured on either side.
- The one EAS Workflow run (`Create Production Builds`, 2026-02-17) failed.

So the most likely outcome of the store steps below is "confirm there is nothing
to remove". Do the confirmation anyway: James may have created the app records
by hand, and an App Store Connect record created but never submitted still
shows up and still blocks the bundle id.

## 1. App Store Connect (Apple)

Sign in at <https://appstoreconnect.apple.com> with the Apple Developer account
that owns `com.fitglue.mobile`.

1. **Does the app exist?** Apps → look for *FitGlue*. If it is not there, also
   check the Apple Developer portal: <https://developer.apple.com/account> →
   *Certificates, Identifiers & Profiles* → *Identifiers* → search
   `com.fitglue.mobile`. An App ID identifier without an App Store Connect
   record is harmless; leave it (deleting identifiers is irreversible and buys
   nothing).
2. **If the app exists, remove it from sale — do not delete it.**
   - Apps → *FitGlue* → *Distribution* tab (older UI: *App Store* tab) →
     *Pricing and Availability* in the left column.
   - Under *App Availability* click **Remove from Sale** (if your UI only shows
     an *Edit* button next to the country list, click it, deselect all
     countries and regions, Save). Confirm.
   - *Remove from sale* keeps the app record, its reviews, its bundle id and its
     SKU, and takes it off the store within a few hours; existing installs keep
     working. It is reversible from the same page.
   - **Delete App** (bottom of *General → App Information*) is the thing *not*
     to click: it is permanent, the SKU can never be reused, and the bundle id
     is locked to the deleted record forever. Nothing we want needs it.
3. **TestFlight.** Apps → *FitGlue* → *TestFlight* tab. For each build listed
   under iOS: open it → **Expire** the build. Then under *Internal Testing* /
   *External Testing* remove the groups (or just remove all testers). If there
   are no builds — expected, see above — skip.
4. **Pending submissions.** If a version is sitting in *Waiting for Review* or
   *In Review*, open it and choose *Remove this version from review* /
   *Cancel submission* first; you cannot remove an app from sale while a
   version is in review.
5. **Do not** cancel the Apple Developer Program membership here. It is a
   separate decision (auto-renews yearly at $99; letting it lapse would also
   pull any remaining app from the store, which is fine, but that is a cost
   decision, not a retirement step).

## 2. Google Play Console

Sign in at <https://play.google.com/console> with the developer account that
owns `com.fitglue.mobile`.

1. **Does the app exist?** *All apps* → look for *FitGlue* /
   `com.fitglue.mobile`. If absent, there is nothing to do on Play; the package
   name is simply unused.
2. **If it exists, unpublish it — do not delete it.**
   - Open the app → left nav *Release* → *Setup* → *Advanced settings* →
     *App availability* tab → select **Unpublished** → *Save*.
   - *Unpublished* is Play's "remove from sale": the listing disappears from
     the store (for new users; existing installs keep working), the package
     name stays reserved to this account, and it is reversible from the same
     screen. Play may take several hours to apply it.
   - **Delete app** is only offered for apps that were never published (it
     appears on the same *Advanced settings* page for drafts). Even then,
     leave it: a deleted package name can never be registered again by anyone.
3. **Testing tracks.** *Release* → *Testing* → *Internal testing* (and
   *Closed testing* if used) → *Testers* tab → remove every tester list. If an
   internal release is still *Available to testers*, unpublishing covers it, but
   removing the testers stops the "you have been invited" link working too.
4. **Pending reviews / declarations.** If a release is *In review* you will be
   blocked from changing availability until it finishes or is discarded
   (*Release* → *Production* / the track → *Discard release*). The Health
   Connect permissions declaration form, if one was submitted, needs no action:
   it only matters for published apps.
5. **Do not** close the developer account. Nothing requires it; it is the thing
   that keeps the package name reserved.

## 3. Expo / EAS (expo.dev, account `ripixel`, project `fitglue-mobile`)

Project page: <https://expo.dev/accounts/ripixel/projects/fitglue-mobile>.

1. **Back up the Android keystore first.** Left nav *Credentials* → *Android*
   → `com.fitglue.mobile` → the keystore → **Download**. Put it (and the
   passwords EAS shows you) in the password manager. If the app was never
   published, this keystore signed only sideloaded APKs and is worthless, but
   it costs nothing to keep and you cannot regenerate it.
2. **Delete the build credentials.** Same *Credentials* page:
   - *Android*: delete the keystore and any FCM / Google service account
     entries listed under the app.
   - *iOS*: delete the distribution certificate, the provisioning profile(s)
     and the push key (APNs) for `com.fitglue.mobile`, if any were ever
     created. Then revoke the matching certificate on Apple's side:
     <https://developer.apple.com/account> → *Certificates, Identifiers &
     Profiles* → *Certificates* → the Apple Distribution cert EAS created →
     *Revoke*; and *Keys* → the APNs key EAS created (named after the project)
     → *Revoke*. Revoking the dist cert only matters if nothing else is signed
     with it; it is per-account, so check the name/creation date before you
     revoke.
3. **Disconnect GitHub.** Project *Settings* → *GitHub* → disconnect the
   `Ripixel-Studio/fitglue-mobile` repository. Then on GitHub:
   <https://github.com/organizations/Ripixel-Studio/settings/installations> →
   *Expo GitHub App* → *Configure* → remove `fitglue-mobile` from the repos it
   can see (or uninstall the app if this was its only repo). This is what the
   deleted `.eas/workflows/create-production-builds.yml` was wired to; without
   the connection no one can trigger a build from a push or from the Expo
   dashboard's *Workflows* tab.
4. **Project secrets / environment variables.** Project *Settings* →
   *Environment variables* — delete anything there (there should be nothing;
   the app only used the public `EXPO_PUBLIC_ENVIRONMENT`).
5. **EAS Update.** *Updates* → branch `production` → delete the branch (and any
   `preview` / `development` branches). This stops any installed APK from
   pulling an OTA update. Not strictly needed since James's phone is the only
   install, but it is one click and it is the closing of a door.
6. **Optionally delete the Expo project** (project *Settings* → *Delete
   project*, bottom of the page). Irreversible, but after steps 1–5 nothing
   depends on it and it removes the last place a build could be launched from.
   The Expo *account* stays; nothing else uses it today, and deleting an account
   is not on the table.

## 4. Fitbit / Strava OAuth redirect URIs — nothing to do

Searched `app.json` and `src/` for a custom URL scheme, `fitglue://`, redirect
URIs, client ids, `openAuthSessionAsync`, Strava or Fitbit SDK usage. There
are none. The app declares no `scheme`, and *Connect Strava / Fitbit* in the
app is just the server's web app running in a WebView
(`src/screens/WebAppScreen.tsx`; the `mobileConnect(provider)` endpoint in
`src/config/api.ts` points at `/api/v2/users/me/integrations/{provider}/connect`
on the server). The OAuth apps and their redirect URIs are registered to
`fitglue.tech` by `fitglue-server`, and James keeps using both sources, so **do
not** touch the Strava or Fitbit developer apps as part of this retirement.

## 5. Things that belong to other tasks (so you know they are covered)

These are server/cloud facts the mobile app leaned on. They are deliberately
*not* in this PR; the task that owns them is in brackets.

- **The mobile ingest path in `fitglue-server` [F1.6].** Precise names, because
  the spec calls it `/webhook/mobile` and the code does not:
  - `POST /users/me/mobile/sync` in `src/go/services/api-client/internal/server/users.go`
    (handler in `mobile_handlers.go`, proto in `src/proto/gateway/client.proto`,
    OpenAPI entry in `docs/api/gateway/client.openapi.yaml`).
  - The `mobile` `SourceProvider` registered in
    `src/go/services/api-webhook/main.go` from
    `internal/webhook/sources/mobile/`.
  - Pub/Sub topic `topic-mobile-activity` (`terraform/pubsub.tf`, resource
    `google_pubsub_topic.mobile_activity`), and the `mobile-sync-handler`
    service name in `terraform/analytics.tf`.
  - The Firebase custom-token "WebView auth bridge" IAM grant in
    `terraform/iam.tf`, the `/web-auth-token` endpoint it served, and the FCM
    push-token registration endpoint — all exist only for this app.
- **Firebase app registrations** in project `fitglue-server-prod` (Android
  `1:605889586984:android:8cfb0…`, iOS `1:605889586984:ios:9375a…`): remove
  under Firebase console → *Project settings* → *Your apps* → *Remove this app*
  once F1.6 has deleted the FCM/custom-token code, not before. The OAuth client
  `605889586984-k8v7…` listed in `google-services.json` is the project's
  Firebase-managed **web** client (`client_type: 3`), which the web app's
  Google sign-in may share — leave it alone; it is not mobile-specific.
- **Sentry project `ripixel-studio/react-native`**: nothing will report to it
  once the APK is uninstalled. Delete it from Sentry whenever the Sentry
  project list is next tidied; no task owns it, it is a one-click removal.
- **The `mobile/v*` git tags**: none exist (no release was ever tagged), so
  there is nothing to preserve.

## 6. Done when

- [ ] App Store Connect: app absent, or present and *Removed from Sale*; no
      TestFlight builds active; developer account untouched.
- [ ] Play Console: app absent, or *Unpublished*; tester lists empty; developer
      account untouched.
- [ ] EAS: keystore backed up, credentials deleted, GitHub disconnected,
      update branches deleted (project deleted or not, James's call).
- [ ] `gh repo archive Ripixel-Studio/fitglue-mobile -y` run after the retiring
      PR merged.
- [ ] James's phone: uninstall the sideloaded FitGlue APK, and remove FitGlue
      from Health Connect's connected apps (*Settings → Health Connect → App
      permissions*) so the OS stops holding a stale grant.
