# 02 · Requirements: Flashlight

*Plain English: what the app must do, written as checks that either pass or fail.*

`Feature: Flashlight | Platform: mobile app (iOS + Android) | Version: 1 | Date: 2026-11-16 | Status: draft`

Terms: **torch** = the phone's built-in LED light. **Screen light** = the app's full-screen white light (SCR2). **Light** = whichever is in use.

## User stories and acceptance criteria

### US1 · Turn the light on (Must · JS1)
As someone suddenly in the dark, I want to turn the light on with one tap, so that I can see straight away.

- **AC1.1** WHEN the user taps the main button while the torch is off and available THE SYSTEM SHALL turn the torch on within 500 ms of the tap.
- **AC1.2** WHEN the torch turns on THE SYSTEM SHALL change the button to its "on" appearance, change its label to "Turn light off", and give a light haptic tap.
- **AC1.3** WHEN the torch turns on THE SYSTEM SHALL announce "Light on" to the screen reader.
- **AC1.4** The main button SHALL cover at least 50% of the screen width and be reachable with one thumb (centre of the screen or lower).
- **AC1.5** IF the user taps the button several times quickly THEN THE SYSTEM SHALL end in the state matching the last tap, with no flicker left over and no error.

### US2 · Turn the light off (Must · JS1, JS3)
As someone who has finished using the light, I want to turn it off with one tap, so that I save battery.

- **AC2.1** WHEN the user taps the main button while the torch is on THE SYSTEM SHALL turn the torch off within 500 ms.
- **AC2.2** WHEN the torch turns off THE SYSTEM SHALL change the button to its "off" appearance, change its label to "Turn light on", give a light haptic tap, and announce "Light off".

### US3 · Get light when the torch can't be used (Must · JS2)
As someone whose phone can't use its torch, I want a screen light instead, so that I'm not left in the dark.

- **AC3.1** IF the device has no torch THEN THE SYSTEM SHALL show the "No flashlight on this device" state (SCR1-unavailable) with a "Use screen light" button, and never show a torch toggle.
- **AC3.2** IF the user has refused the permission needed to control the torch THEN THE SYSTEM SHALL show the permission-denied state (SCR1-denied) with "Use screen light" and "Open Settings" buttons.
- **AC3.3** IF turning the torch on fails (busy, too hot, low power or unknown error) THEN THE SYSTEM SHALL keep the button in the "off" state, show the matching message from 03 (SCR1-error), and offer "Try again" and "Use screen light".
- **AC3.4** WHEN the screen light opens THE SYSTEM SHALL fill the screen with white, raise screen brightness to maximum, and keep the screen awake. IF the app can't change brightness THEN THE SYSTEM SHALL still show white and show the hint "Turn up your screen brightness for more light" for 3 s.
- **AC3.5** WHEN the user taps anywhere on the screen light, or uses the system back gesture or button, THE SYSTEM SHALL close it within 300 ms, restore the screen brightness the user had before, and let the screen sleep normally again.
- **AC3.6** IF the app is sent to the background while the screen light is open THEN THE SYSTEM SHALL restore the user's previous brightness.

### US4 · Always know whether the light is on (Must · JS3)
As someone who pockets their phone or switches apps, I want the app to show whether the light is really on, so that I don't drain my battery by accident.

- **AC4.1** WHILE the app is open THE SYSTEM SHALL show the torch's actual state, including when it was switched on or off from outside the app (e.g. Control Center or Quick Settings), updating within 1 s of the change.
- **AC4.2** WHEN the user returns to the app THE SYSTEM SHALL show the torch's actual state within 300 ms.
- **AC4.3** WHEN the app goes to the background or the phone is locked while the torch is on: WHERE the platform allows the torch to stay on THE SYSTEM SHALL leave it on; IF the platform turns it off THEN THE SYSTEM SHALL show the "off" state on return and show the hint "Your phone turned the light off when you left the app" for 3 s. (Which applies on each platform is OQ1.)
- **AC4.4** WHEN the app is closed fully (swiped away) THE SYSTEM SHALL NOT leave the torch on without a way to see it in the app. [Assumption: the platform turns it off]

### US5 · Trust that it's private (Must · JS4)
As someone installing a simple utility, I want no ads, no account and no tracking, so that I can trust the app.

- **AC5.1** THE SYSTEM SHALL make no network requests at any time. (Check: run the app through FL1–FL3 with a network monitor; zero requests.)
- **AC5.2** THE SYSTEM SHALL show no ads and contain no advertising, analytics or tracking SDKs. (Check: dependency audit.)
- **AC5.3** THE SYSTEM SHALL work with no account and no sign-in.
- **AC5.4** THE SYSTEM SHALL ask only for the permissions strictly needed to control the torch, and only when the user first taps the button, never at launch.
- **AC5.5** WHEN a permission is needed THE SYSTEM SHALL show the explanation in SCR1-permission before the system prompt.

### US6 · Open with the light already on (Should · JS1) *(Later, not in v1)*
As someone in a hurry, I want the light to come on as soon as I open the app, so that I need zero taps.

- **AC6.1** WHERE "Turn on at launch" is enabled THE SYSTEM SHALL turn the torch on within 1.5 s of a cold start.
- Note: needs a settings screen; out of v1 scope.

## Edge cases → criteria
| Edge case | Covered by |
|---|---|
| EC1 Device has no torch (some tablets, older phones) | AC3.1 |
| EC2 Torch busy (camera app or another app using it) | AC3.3 |
| EC3 Phone too hot or in low-power mode, torch disabled by the system | AC3.3 |
| EC4 Torch switched on/off from Control Center / Quick Settings while app open | AC4.1 |
| EC5 App backgrounded or phone locked with torch on | AC4.3 |
| EC6 Rapid repeated taps | AC1.5 |
| EC7 Screen light open when app is backgrounded | AC3.6 |
| EC8 Permission refused, or later revoked in Settings | AC3.2, FF1 |
| EC9 Permission granted later in Settings, user returns | AC4.2 + FF1 step 5 |

## Non-functional requirements
- **NFR1 Speed:** torch on or off within 500 ms of a tap (AC1.1, AC2.1), measured on a mid-range phone from each platform.
- **NFR2 Start-up:** main screen interactive within 1.5 s of a cold start on a mid-range phone.
- **NFR3 Accessibility:** WCAG 2.2 AA. Every control has a screen-reader label and announces state changes; touch targets ≥ 44×44 pt (iOS) / 48×48 dp (Android); text contrast ≥ 4.5:1 in every state including the white screen light; supports system text sizes up to the largest setting without clipping; works in light and dark mode; respects "reduce motion" (no animations over 200 ms when on).
- **NFR4 Privacy:** no personal data is collected, stored or transmitted (AC5.1–AC5.3). Store privacy declarations: "Data Not Collected" / "No data collected".
- **NFR5 Measurement:** no in-app analytics events. Success is read from App Store Connect and Google Play Console (installs, retention, rating, crash-free sessions).
- **NFR6 Reliability:** crash-free sessions ≥ 99.5% (store consoles). The app never leaves the UI showing a different state from the torch for more than 1 s (AC4.1).
- **NFR7 Battery:** the app does no background work. The screen stays awake only while the screen light is open.
- **NFR8 Offline:** every feature works with no connection (there are no online features).

## Traceability
| Job story | User story | Acceptance criteria | Screens / flows |
|---|---|---|---|
| JS1 | US1 | AC1.1–AC1.5 | SCR1 · FL1, FL2 |
| JS1, JS3 | US2 | AC2.1–AC2.2 | SCR1 · FL2 |
| JS2 | US3 | AC3.1–AC3.6 | SCR1, SCR2 · FL3, FF1–FF3 |
| JS3 | US4 | AC4.1–AC4.4 | SCR1 · FL4, FF4 |
| JS4 | US5 | AC5.1–AC5.5 | SCR1 · FL1 |
| JS1 | US6 (later) | AC6.1 | n/a in v1 |
