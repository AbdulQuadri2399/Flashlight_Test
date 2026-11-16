# 01 · PRD: Flashlight

*Plain English: why this exists, who it's for, what's in and out, and how we'll know it worked.*

`Feature: Flashlight | Platform: mobile app (iOS + Android) | Version: 1 | Date: 2026-11-16 | Status: draft`

## 1. Overview
*What we're building, in one paragraph.*

A minimal flashlight app: one large on/off button, a full-screen white "screen light" when the phone's light can't be used, and an always-accurate on/off state. No ads, no account, no data collected. Recommendation: **Build, as a pipeline test run** (confidence: high that it's buildable; low that it changes behaviour, see §2). Owner: product owner (user). Status: draft.

## 2. Problem & evidence
*What's going wrong for people, and how sure we are.*

- **Problem:** people need light instantly in the dark, and the alternatives are either buried in system menus or are third-party flashlight apps that have a reputation for ads and excessive permissions. [Assumption]
- **Who:** smartphone owners who suddenly need light: power cuts, finding keys in a bag, walking outside at night, looking under furniture.
- **Evidence rating: assumed.** Both iOS (Control Center, lock screen) and Android (Quick Settings) already include a flashlight, so the core need is largely met by the phone itself. Evidence for *this* app's value is weak; that is acceptable because v1's main purpose is to test the build pipeline.
- **Cost of doing nothing:** none for users; for us, the pipeline goes untested.
- **Why now:** internal: we need a small, real product to run through the product-to-build pipeline. [Assumption: no external urgency]

## 3. Target user & JTBD
*The people, their moment, and what they do today.*

- **JS1:** **When** I'm suddenly in the dark (power cut, keys in a bag, outside at night) **I want to** get light immediately without thinking **So I can** see what I'm doing or where I'm going.
- **JS2:** **When** my phone can't use its light (no torch, I refused access, it's too hot) **I want to** still get some light **So I'm not** left in the dark.
- **JS3:** **When** I lock my phone, pocket it or switch apps **I want to** know whether the light is still on **So I don't** drain my battery without realising.
- **JS4:** **When** I install a simple utility **I want to** know it isn't tracking me or showing me ads **So I can** trust it on my phone.

**Current workarounds:** the built-in system flashlight (swipe into Control Center / Quick Settings, or the lock-screen button); third-party flashlight apps; holding up a phone screen on a white image.

## 4. Outcome & success metrics
*The behaviour we want, and the numbers that show it.*

- **Target behaviour:** people who install it come back to use it when they need light, instead of the built-in control. [Assumption]
- **Primary metric:** Day-30 retention ≥ 20% of installers (from App Store Connect / Google Play Console; no in-app analytics). Baseline: none (new app).
- **Guardrails:** store rating ≥ 4.5; crash-free sessions ≥ 99.5% (store consoles); zero privacy complaints or rejections.
- **Early signs:** Day-1 retention ≥ 40%; reviews mentioning "fast" or "simple".
- **Quality targets** (checked in testing, see NFR1–NFR2): light on within 0.5 s of a tap; app ready to use within 1.5 s of a cold start on a mid-range phone.

## 5. Solution
*What we'll build, and why this shape.*

- **Chosen approach:** a single-purpose app whose whole screen is the control. One big button toggles the light. If the light can't be used, the same screen offers a white screen light.
- **Behavioural mechanism:**
  - *Ability (Fogg):* removes steps and decisions: one target, impossible to miss, usable half-awake in the dark.
  - *Trust:* "no ads, no data" removes a known reason people avoid flashlight apps.
  - *Feedback:* the button, label, screen-reader announcement and a haptic tick all confirm the state, so people never wonder whether it worked (JS3).
- **Alternatives considered:**
  - *Don't build; point people to the built-in flashlight:* the honest best answer for users, but it doesn't test the pipeline.
  - *Feature-rich flashlight (SOS, strobe, colours, brightness):* more to test and more to go wrong; strobe adds seizure risk. Deferred.
  - *Web app:* rejected. Browsers on iPhone can't switch the phone's light on, so half of users would only get the screen light.

## 6. Scope & non-goals
*What's in now, what's later, and what we're deliberately not doing.*

- **v1 (Must):** on/off toggle (US1, US2), screen-light fallback (US3), accurate light state (US4), no ads / accounts / data (US5).
- **Later (Should/Could):** open with the light already on (US6, Should); brightness levels; SOS signal; coloured screen light; auto-off timer.
- **Non-goals:** strobe (seizure risk), home-screen widgets, ads, accounts, in-app analytics, monetisation, tablets as a design target (they still work via the screen light).

## 7. Constraints for engineering
*Limits the build must respect. Constraints only, no technology decisions.*

- Platforms: iOS and Android phones, current and previous major OS versions. [Assumption]
- **Privacy:** no third-party analytics, advertising or tracking SDKs. The app makes **no network requests**. Crash data comes only from the stores' built-in reporting. Store privacy labels must read "Data Not Collected" (iOS) and "No data collected" (Google Play).
- **Permissions:** request only what is strictly needed to control the light. [Assumption] The platform may require camera permission to control the light; if so, ask only at the moment of first use, with an explanation first (see 03, SCR1 permission states).
- Fully usable offline (it has no online features).
- No account, no sign-in.
- Budget/deadline: none set. Preference: keep the codebase small enough to read in one sitting.

## 8. Risks, assumptions & planned tests
*What could sink it, and how we'll find out cheaply.*

**Four risks**
| Risk | Rating | Why |
|---|---|---|
| Will people want it? | High | Built-in flashlights already do the job |
| Can they use it easily? | Low | One button |
| Can we build it? | Low–Med | Simple, but light control differs by platform (see OQ1–OQ3) |
| Does it work for the business? | Low | No revenue expected; it's a pipeline test |

**Riskiest assumptions**
| # | Assumption | Matters if wrong | Evidence | Test | Pass line |
|---|---|---|---|---|---|
| A1 | People will choose this over the built-in flashlight | High | None | Small store launch; read retention in store consoles | Day-30 retention ≥ 20% |
| A2 | The light can stay on while the app is in the background or the phone is locked | Med | None | Device test on 1 iPhone + 1 Android (OQ1) | Light stays on for 5 min with screen locked |
| A3 | Camera permission is (or isn't) required to control the light | Med | Partial | Device test during tech spec (OQ2) | Behaviour documented in 04 |

No fake-door or A/B test is needed in the product.

**Pre-mortem** ("it's 6 months later and this flopped")
1. Nobody switches from the built-in flashlight → accepted; v1 is a pipeline test. Revisit only if Day-30 retention is surprisingly high.
2. The permission prompt scares people off → explain first, in plain words, and offer the screen light if they say no.
3. The light silently turns off when the phone locks, so people think it's broken → always show the true state on return (US4) and say clearly what happens when you leave the app.

**Assumption log**
| Assumption | Confidence | How to test |
|---|---|---|
| Need for instant light is the motivation (JS1) | Med | Store reviews |
| Third-party flashlight apps are distrusted | Med | Reviews of competitors |
| Camera permission may be required | Med | Device test (OQ2) |
| Light can stay on in the background | Low | Device test (OQ1) |
| Current + previous OS versions are enough | Med | Store console device data |

## 9. Rollout & measurement
*How it gets released, and how we judge it.*

- **Release:** internal testing (TestFlight / Play internal testing) on at least 1 iPhone and 1 Android phone → public store release.
- **Measurement:** store console data only (installs, Day-1/7/30 retention, rating, crash-free sessions), because the app collects no data. There are no in-app analytics events (see NFR5).
- **Review:** 30 days after public release.
- **Decide:** Day-30 retention ≥ 20% and rating ≥ 4.5 → consider Later features. Below that → keep as is; the pipeline test is complete either way.

## 10. Open questions
*Unknowns, who resolves them, and whether they block building.*

| ID | Question | Resolved by | Blocks building? |
|---|---|---|---|
| OQ1 | Can the light stay on while the app is in the background or the phone is locked, on both platforms? | Device test in tech spec | No: 02 defines behaviour for both outcomes (AC4.3) |
| OQ2 | Does controlling the light require camera permission on each platform? | Device test in tech spec | No: 03 covers both cases |
| OQ3 | Can the app maximise and then restore screen brightness for the screen light on both platforms? | Tech spec | No: AC3.4 has a fallback |
