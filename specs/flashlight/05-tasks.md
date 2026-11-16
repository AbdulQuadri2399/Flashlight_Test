# 05 · Tasks: Flashlight

*Plain English: the build, in the order it happens. Each slice ends with something you can try on a phone.*

`Feature: Flashlight | Platform: mobile app (iOS + Android) | Version: 1 | Date: 2026-11-16 | Status: draft`

Sizes: S = under an hour or two, M = half a day. `[P]` = can run in parallel with the task above. Stop after every slice check for review.

## Slice 0: Walking skeleton
Goal: an empty app builds, runs on both phones and passes CI.
You'll be able to see: a dark screen reading "Flashlight" running on your iPhone and Android phone from a development build.

- [ ] T0.1 Scaffold Expo SDK 58 app with Expo Router, TypeScript strict, pnpm; add lint, format, typecheck, test scripts — S — Refs: 04 §2 — Depends on: none
  Done when: `pnpm lint && pnpm typecheck && pnpm test` all exit 0.
- [ ] T0.2 `app.config.ts`: portrait only, automatic light/dark, `updates.enabled: false`, bundle IDs, iOS privacy manifest (no tracking, no data collected, required-reason APIs), blocked permissions driven by `APP_VARIANT` — S — Refs: D3, D7, D8, AC5.1, 04 §10 — Depends on: T0.1
  Done when: `npx expo config --type public` shows these values; `APP_VARIANT=production npx expo prebuild --clean --platform android && grep -E 'INTERNET|CAMERA' android/app/src/main/AndroidManifest.xml` prints nothing; `APP_VARIANT=development` run of the same shows INTERNET present; `ios/*/PrivacyInfo.xcprivacy` exists after `APP_VARIANT=production npx expo prebuild --clean --platform ios` with `NSPrivacyTracking` false and an empty collected-data list.
- [ ] T0.3 [P] `scripts/audit-deps.mjs`: read `pnpm list --prod --depth Infinity --json` and fail on an exact denylist of package names and scopes (`@sentry/*`, `@react-native-firebase/*`, `firebase`, `@segment/*`, `@amplitude/*`, `mixpanel-react-native`, `react-native-google-mobile-ads`, `react-native-fbsdk-next`, `expo-updates`, `expo-insights`, …), not substrings — S — Refs: AC5.2, NFR4, NFR5 — Depends on: T0.1
  Done when: `pnpm audit:deps` passes; adding `@sentry/react-native` temporarily makes it fail.
- [ ] T0.4 [P] GitHub Actions CI: install, lint, typecheck, test, audit:deps on every push — S — Refs: 04 §11 — Depends on: T0.3
  Done when: a pushed commit shows a green CI run.
- [ ] T0.5 EAS: `eas.json` with `development`, `preview`, `production` profiles; development builds for iOS and Android — M — Refs: R2 — Depends on: T0.2
  Done when: the dev build installs and opens on 1 iPhone + 1 Android; a `preview` Android build (no INTERNET) also launches (R1).
- [ ] T0.6 [P] `CLAUDE.md` at project root (product line, link to `specs/flashlight/`, stack + link to 04, exact commands, rules: slice by slice, tick tasks only after their check passes and show output, no network/analytics code, tests never inside `app/`, update 04 when a decision changes) — S — Refs: 04 §2, §11 — Depends on: T0.1
  Done when: file exists and is under 40 lines; `pnpm install && pnpm lint && pnpm typecheck && pnpm test && pnpm audit:deps` (the commands it lists) all exit 0.
- [ ] T0.7 [P] `e2e/DEVICE-CHECKLIST.md` with empty tables for both phones: S1 spike (locked 5 min, permission needed, error per scenario, external toggle ≤ 1 s), S2 brightness restore, timings (tap→light ×10 ≤ 500 ms, cold start ≤ 1.5 s, SCR2 close ≤ 300 ms, state on return ≤ 300 ms), AC4.4 swipe-away, accessibility (VoiceOver/TalkBack reads every control and announces on/off in FL1–FL4; largest text size no clipping on each SCR1/SCR2 state; light and dark mode; reduce motion on, no animation > 200 ms), privacy (zero requests FL1–FL3; airplane-mode FL1–FL3), busy and low-power errors — S — Refs: NFR1–NFR3, NFR8, AC3.5, AC4.2, AC4.4, AC5.1 — Depends on: T0.1
  Done when: the file exists with every row above, each with a pass/fail column per phone.

Slice check: CI green; dev build running on both phones; release Android manifest has no INTERNET permission (or D7 fallback recorded in 04).

## Slice 1: Torch spike and native module (riskiest technical unknowns first)
Goal: the app can switch the real torch and hear about outside changes; OQ1 and OQ2 get answered.
You'll be able to see: a plain test button that turns the real flashlight on and off, and its label changing when you use Control Center / Quick Settings.

- [ ] T1.1 Create `modules/torch` Expo Module skeleton with typed JS API O1–O5 and a Jest mock — S — Refs: 04 §5 — Depends on: T0.5
  Done when: `pnpm typecheck` passes; a Jest test imports the mock and calls every method.
- [ ] T1.2 Android: `CameraManager.setTorchMode`, `TorchCallback` → `onTorchChange`, error mapping, `getCapabilities` — M — Refs: AC1.1, AC2.1, AC3.3, AC4.1, EC1–EC4 — Depends on: T1.1
  Done when: on the Android phone, the test button toggles the torch; a Quick Settings toggle changes the label within 1 s.
- [ ] T1.3 [P] iOS: `AVCaptureDevice` torch on/off, KVO on `torchActive`/`isTorchAvailable` → `onTorchChange`, error mapping, `getCapabilities` — M — Refs: same as T1.2 — Depends on: T1.1
  Done when: same check on the iPhone with Control Center.
- [ ] T1.4 Spike S1 on both phones; write results into 04 (§5 error mapping, D9 permission flag, `osTurnsOffInBackground` per platform in §7, §12) and answer OQ1/OQ2 in 01 §10 — S — Refs: OQ1, OQ2, A2, A3, AC4.3 — Depends on: T1.2, T1.3, T0.7
  Done when: `e2e/DEVICE-CHECKLIST.md` has a filled S1 table (stays on locked 5 min? permission needed? error per scenario? event ≤ 1 s?) for both platforms, and 04 is updated.
- [ ] T1.5 Backstop: turn torch off when the app process is destroyed — S — Refs: AC4.4 — Depends on: T1.4
  Done when: AC4.4 row of the device checklist filled for both phones (torch on → swipe app away → torch off?). If the torch survives on a platform, record it and confirm the app shows "on" when reopened (best-effort backstop, 04 §8).

Slice check: device checklist S1 + AC4.4 filled for both phones. **Review stop:** confirm OQ1/OQ2 answers before building the UI on top.

## Slice 2: Turn the light on and off (US1, US2)
Goal: the real SCR1 with the big button, all feedback, and rapid-tap safety.
You'll be able to see: SCR1 in **off**, **turning on** and **on** states, with haptic tap and VoiceOver/TalkBack saying "Light on"/"Light off".

- [ ] T2.1 `reducer.ts` + tests: off/turningOn/on/turningOff, `desired` handling for rapid taps, external events — M — Refs: AC1.5, AC4.1, EC4, EC6, FF5, NFR6 — Depends on: T1.1
  Done when: `pnpm test reducer` passes, incl. cases "5 rapid taps ends at last tap", "external off while on", "confirm arrives after another tap".
- [ ] T2.2 [P] `theme.ts` + `copy.ts` (all text from 03) + contrast unit test — S — Refs: NFR3, 03 microcopy — Depends on: T0.1
  Done when: `pnpm test theme` proves every text/background pair ≥ 4.5:1 in light and dark system themes and on white.
- [ ] T2.3 `TorchProvider`: wires reducer to O3/O5, single in-flight call, haptic + announcement on confirmed change only — M — Refs: AC1.1–AC1.3, AC2.1–AC2.2, AC1.5 — Depends on: T2.1
  Done when: RNTL tests with mock: tap → `setTorch(true)` called once, one haptic, `announceForAccessibility('Light on')`; tap again → `setTorch(false)`, one haptic, `announceForAccessibility('Light off')`; external `onTorchChange` → state, haptic and announcement update; error path calls no haptic; on Android the haptic call is `performAndroidHapticsAsync`.
- [ ] T2.4 `MainButton` + `StatusArea` + SCR1 layout: ≥ 50% width, centre/lower, on/off appearance, labels, roles, spinner only after 300 ms — M — Refs: AC1.2, AC1.4, AC2.2, SCR1 off/turning on/on, NFR3 — Depends on: T2.2, T2.3
  Done when: `pnpm test index` passes (labels "Turn light on"/"Turn light off", status lines, `accessibilityRole`, width ≥ 50% of window, button centre at or below the screen's vertical midpoint, no spinner before 300 ms); screenshots of off/on at default and largest text size show no clipping.
- [ ] T2.5 Maestro FL2 — S — Refs: FL2, AC5.3 — Depends on: T2.4
  Done when: `maestro test e2e/fl2-everyday.yaml` passes on the dev build.
- [ ] T2.6 Timing check — S — Refs: AC1.1, AC2.1, NFR1, NFR2 — Depends on: T2.4, T0.7
  Done when: device checklist shows 10/10 taps ≤ 500 ms and cold start ≤ 1.5 s on each phone.

Slice check: FL2 passes; reducer + SCR1 tests green; timing recorded. **Review stop.**

## Slice 3: Screen light and failure states (US3)
Goal: a way to light from every dead end.
You'll be able to see: SCR2 (white, max brightness, "Tap anywhere to close"), and SCR1 in **unavailable**, **error: busy**, **error: too hot / low power**, **error: unknown**.

- [ ] T3.1 SCR2 `screen-light.tsx`: white, status bar hidden, save → max brightness → keep awake; restore on close/background; re-apply on return; 3 s hints; tap anywhere / Android back closes — M — Refs: AC3.4–AC3.6, SCR2 on / brightness locked, EC7, NFR7, S2 — Depends on: T0.5, T0.7
  Done when: `pnpm test screen-light` passes (brightness saved/restored on unmount and on `background`, keep-awake released, hint shown when `setBrightnessAsync` throws, hint disappears after 3 s); device checklist S2 shows brightness restored on both phones.
- [ ] T3.2 SCR1 **unavailable** (no toggle; same-size "Use screen light") — S — Refs: AC3.1, EC1 — Depends on: T2.4, T3.1
  Done when: RNTL with `hasTorch=false`: no toggle rendered, "No flashlight on this device" shown, button navigates to SCR2.
- [ ] T3.3 [P] SCR1 error states busy / too hot / unknown with "Try again" (one attempt per tap) and "Use screen light"; button stays off; no haptic; failed turn-off keeps **on** and shows the Q1 copy — M — Refs: AC3.3, EC2, EC3, FF2, FF3, FF6, NFR3 — Depends on: T2.4
  Done when: RNTL test per error code shows the exact 03 copy and both buttons, each ≥ 48×48; "Try again" calls `setTorch` exactly once; reducer test "turn-off fails → state stays on, error line shown".
- [ ] T3.4 Maestro FL3 + device checklist for real busy (camera app open on Android) and low power mode — S — Refs: FL3, FF2, FF3 — Depends on: T3.1–T3.3
  Done when: `maestro test e2e/fl3-screen-light.yaml` passes; checklist rows filled for busy, low power and SCR2 close ≤ 300 ms.

Slice check: FL3 passes; every SCR1 error/unavailable state and both SCR2 states have screenshots. **Review stop.**

## Slice 4: Always-true state when leaving and returning (US4)
Goal: the app never lies about the light.
You'll be able to see: lock the phone with the light on, come back, and the app shows the right state (with the "Your phone turned the light off…" hint if the OS turned it off).

- [ ] T4.1 `AppState` handling: record `wasOnWhenLeft`; on `active` call O2 and update within 300 ms; show the 3 s hint only if turned off while away **and** `osTurnsOffInBackground` is true for this platform (04 §7) — S — Refs: AC4.2, AC4.3, EC5, FF4 — Depends on: T2.3, T1.4
  Done when: reducer/RNTL tests: "left on, returns off, OS turns off → hint shown 3 s then hidden", "left on, returns off, OS keeps on → no hint", "left on, returns on → no hint".
- [ ] T4.2 Maestro FL4 (background → foreground) + device checklist: external toggle while open ≤ 1 s, locked 5 min — S — Refs: FL4, AC4.1, AC4.3, EC4 — Depends on: T4.1
  Done when: `maestro test e2e/fl4-leave-return.yaml` passes; checklist rows (external toggle ≤ 1 s, state on return ≤ 300 ms, locked 5 min) filled for both phones.

Slice check: FL4 passes; device checklist US4 rows complete. **Review stop.**

## Slice 5: Permission flow (US5 AC5.4–5.5) — conditional on spike S1
Goal: ask for permission only if a platform needs it, explained first.
You'll be able to see: (if needed) the "One quick permission" card, the system prompt, and the **permission denied** state with "Open Settings".

- [ ] T5.1 SCR1 **permission needed** card + **permission denied** state, driven by `permissionRequired` / O2 / O4; never called at launch — M — Refs: AC3.2, AC5.4, AC5.5, EC8, FF1 — Depends on: T2.4, T1.4
  Done when: RNTL with mock `permissionRequired=true`: no `requestPermission` at mount; first tap shows card; "Continue" calls O4; `denied` → greyed button, "Open Settings" calls `Linking.openSettings`; "Continue", "Use screen light instead" and "Open Settings" each ≥ 48×48. With `permissionRequired=false`: card never renders. (Built either way, per D9.)
- [ ] T5.2 Re-check permission on return from Settings — S — Refs: EC9, FF1 step 5 — Depends on: T5.1, T4.1
  Done when: RNTL: denied → app `active` with `granted` → **off** state.
- [ ] T5.3 Only if S1 says a platform needs it: add camera permission + iOS usage string from 03; Maestro FL1 — S — Refs: FL1, 03 platform notes — Depends on: T5.1
  Done when: `maestro test e2e/fl1-first-use.yaml` passes on that platform; or this task is marked "not needed (S1)" with the S1 result linked.

Slice check: permission tests green; FL1 passes or is recorded as not applicable. **Review stop.**

## Slice 6: Hardening and release
Goal: prove the quality and privacy promises, then ship.
You'll be able to see: TestFlight and Play internal-testing builds, and a filled release checklist.

- [ ] T6.1 Accessibility pass: VoiceOver + TalkBack through FL1–FL4; largest text size; light and dark mode; reduce motion on — M — Refs: NFR3, AC1.3, AC2.2 — Depends on: Slices 2–5
  Done when: every accessibility row defined in T0.7 is marked pass for both phones, with screenshots of each SCR1/SCR2 state at largest text size.
- [ ] T6.2 [P] Privacy check: run FL1–FL3 on a release build behind a network monitor (proxy or Xcode Instruments / Android `dumpsys netstats`) — S — Refs: AC5.1, NFR4, NFR8 — Depends on: Slices 2–5
  Done when: checklist records zero requests on both platforms; airplane-mode run of FL1–FL3 passes.
- [ ] T6.3 [P] Performance re-check on release builds — S — Refs: NFR1, NFR2 — Depends on: Slices 2–5
  Done when: same timing checklist as T2.6 passes on release builds.
- [ ] T6.4 Store listing: privacy labels "Data Not Collected" / "No data collected", listing text "One tap for light. No ads, no account, no data collected.", no strobe mention, screenshots — S — Refs: 01 §7, 03 store requirements — Depends on: T6.2
  Done when: App Store Connect shows "Data Not Collected" and Play Console Data safety shows "No data collected"; listing text matches 03 exactly; a search of the listing for "strobe" finds nothing; an EAS production iOS build is accepted by App Store Connect processing (privacy manifest OK).
- [ ] T6.5 EAS production builds → TestFlight + Play internal testing → install on 1 iPhone + 1 Android — S — Refs: 01 §9 — Depends on: T6.1–T6.4
  Done when: both internal builds install and pass FL2 by hand.

Slice check: release checklist complete; internal builds on both phones. **Review stop** before public release.
