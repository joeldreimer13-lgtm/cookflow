# CookFlow for iPhone and Apple Watch

CookFlow has an iPhone companion containing the complete offline shopping and cooking interface, and a native Watch app with store lists, cooking sections, checkboxes, counts, and percentage bars.

## What is ready

- An Xcode project with an iPhone app and an embedded, single-target watchOS app.
- The existing CookFlow interface bundled in the iPhone app, without a web server, accounts, or OpenAI dependency.
- Automatic paired-device syncing through Apple's Watch Connectivity framework.
- Persisted Watch changes while disconnected; explicit, idempotent checkbox updates when connection returns.
- A portable backup format for moving existing Safari/Home Screen lists into the companion app.
- Swift package tests covering offline edits, conflict handling, deleted items, duplicate delivery, acknowledgements, and backup round trips.

The Watch is for viewing lists and checking items off. Add, remove, move, and edit stores/items and paste recipes using the iPhone companion.

## Install on your devices

1. Use a Mac and an Xcode version that supports the OS versions on your iPhone and Watch. Current Xcode 27 requires an M1-or-newer Mac and macOS 26.6 or later; an older Intel Mac cannot run that version. See [Apple's current Xcode requirements](https://developer.apple.com/xcode/system-requirements) and [Xcode in the Mac App Store](https://apps.apple.com/us/app/xcode/id497799835).
2. Open `CookFlowApple.xcodeproj` in Xcode. The project is already generated; Python and third-party project generators are not needed to open it.
3. In Xcode Settings → Apple Accounts, sign in with your Apple Account. In Signing & Capabilities, enable automatic signing and select the same Team for **both** CookFlow and CookFlowWatch targets.
4. If Xcode says the app identifier is unavailable, edit `COOKFLOW_BASE_BUNDLE_ID` in `Config/Signing.xcconfig` to a unique value. The Watch companion identifier follows this value automatically.
5. Connect and trust your iPhone. Enable Developer Mode when Apple requests it. Keep your Watch paired with that iPhone and unlocked.
6. Select the **CookFlow** scheme, choose your iPhone as the run destination, and click Run. Then select **CookFlowWatch**, choose your paired Watch, and click Run. You can also install the embedded Watch app through the Watch app on iPhone after the phone app is installed.
7. Open CookFlow on both devices once, then tap **Sync now** on Watch.

Personal Team provisioning is temporary: Apple's free profiles expire after seven days and require rebuilding/reinstalling. TestFlight distribution requires Apple Developer Program/App Store Connect setup and a signed build; this repository does not contain signing certificates or claim to be an installable App Store/TestFlight release.

## Bring your saved Safari or Home Screen lists

1. In your existing CookFlow website/Home Screen app, open **Move or back up your lists** near the bottom.
2. Tap **Copy for companion**. In the native iPhone CookFlow, tap the Paste button in the top-right toolbar. Apple may ask you to confirm pasting.
3. Alternatively, tap **Save lists**, then **Download backup file**, save `CookFlow-backup.cookflow` in Files, and use the native app's import icon to choose that file.
4. Confirm the import. It replaces the companion app's lists and sends them to Watch.

After migration, use the native iPhone companion for automatic Watch sync. Safari and Home Screen websites cannot access a native app's storage, and this version does not claim live syncing from those separate browser copies. Export a backup from the native iPhone app and import it into Safari when you want to move data back manually.

## Sync behavior

- The iPhone keeps the authoritative list structure and assigns revisions.
- Watch changes save locally before appearing as pending, and update the Watch UI immediately.
- The Watch sends explicit `done = true/false` commands, not toggles. Duplicate delivery cannot reverse a checkbox.
- Changes to different items merge. A newer iPhone change to the same checkbox wins over an older offline Watch change. A short notice appears on Watch when this happens.
- Removed items are not recreated by a queued Watch update. Replacing lists from a backup creates a new document identity, retiring commands for the previous lists.
- Apple controls background delivery timing. **Sync now**, with both apps open and devices connected, is the quickest retry; no guaranteed instant delivery is claimed while apps are closed.
- Small snapshots use application context and immediate messages. Large lists use background file transfer rather than exceeding dictionary payload limits.
- No lists, checkmarks, or device identifiers are sent to a third-party server.

## Checks

```sh
swift test --package-path Core
xcodebuild -project CookFlowApple.xcodeproj -scheme CookFlow -configuration Debug -destination 'generic/platform=iOS Simulator' -derivedDataPath Build CODE_SIGNING_ALLOWED=NO build
xcodebuild -project CookFlowApple.xcodeproj -scheme CookFlowWatch -configuration Debug -destination 'generic/platform=watchOS Simulator' -derivedDataPath Build CODE_SIGNING_ALLOWED=NO build
```

Simulator compilation does not sign or install the app on your physical devices. Paired-device transport, background delivery, and provisioning still require device testing.

`scripts/generate_project.py` is included only to reproduce the checked-in project and Info.plist files. App icons are standard asset catalogs. `--preview` is a local simulator launch argument that shows the bundled public example without connecting to a phone; `--shopping` opens the stores screen and `--items` opens the first store for visual checks. Preview mode does not replace saved device lists or send sample data to a paired iPhone.

## Apple references

- [Running on simulated or physical devices](https://developer.apple.com/documentation/xcode/running-your-app-on-simulated-or-physical-devices)
- [Personal Teams and provisioning](https://developer.apple.com/help/account/basics/about-your-developer-account)
- [Watch Connectivity delivery methods](https://developer.apple.com/documentation/watchconnectivity/transferring-data-with-watch-connectivity)
