# Duress-Aware Wallet Security Standard: A Feature Proposal

> **Phone-friendly readers:** open `duress-aware-wallet-notion-part1.html` / `part2.html` (or `duress-aware-threat-model-matrix.html` for the matrix alone). This Markdown source is unchanged.


**Audience:** Wallet engineers, neobank / card-partner teams, security reviewers  
**Status:** Feature-layer proposal (not a rigid protocol)  
**Examples limited to:** EtherFi, RedotPay, Cast, Tornado Cash  

---

## 1. Executive summary

Self-custodial crypto wallets give users control of keys but almost none of the operational controls people expect from banks: withdrawal limits, cool-downs, trusted-approver workflows, or a credible response to physical coercion. Face ID and device biometrics unlock the real vault for anyone who can force the phone into the owner’s face. Once funds leave via a mixer (historically exemplified by **Tornado Cash**), recovery is effectively gone.

This proposal defines a **duress-aware feature layer**—not one mandatory UI or one shared secret scheme—that wallets and neobanks can adopt in pieces. The core idea: the same account identity can open either a **real vault** or a **decoy vault**, selected by which password is entered after (or instead of) biometric unlock. Outside a home geofence, high-value transfers get time-locks and/or multi-sig approval from a trusted contact. The wallet UI can be masked as a mundane app so the real product is not obvious under shoulder-surfing.

Ship this as an **adoptable vocabulary and policy kit**. Uniform, identical behavior across every wallet is predictable and therefore attackable; a minimal shared language lets partners integrate without freezing design into one brittle standard.

---

## 2. Problem statement

### 2.1 What banks have that wallets lack

Retail banking and card rails routinely expose:

- Per-day / per-transfer limits  
- Delayed settlement or hold periods for unusual amounts  
- Freeze / dispute paths and some fund-tracing on regulated rails  

Self-custodial wallets typically expose:

- Immediate, final settlement once the signature is broadcast  
- No issuer-side “undo”  
- Biometric unlock that maps 1:1 to full spending authority  

Products such as **EtherFi** (self-custody / DeFi-adjacent UX) and card/neobank bridges such as **RedotPay** sit closer to everyday spend—and therefore closer to coercion scenarios—than cold storage. Any duress story must work across both “hot wallet” and “spend rail” surfaces.

### 2.2 The coercion stack

A realistic attack combines:

1. **Physical coercion** (threat of violence).  
2. **Face / biometric unlock** that opens the real balance.  
3. **Immediate transfer**, optionally through a mixer or privacy pool (historically associated with **Tornado Cash**-style obfuscation), after which tracing and recovery collapse.

There is no bank chargeback on L1. Without a deliberate duress mode, the victim’s only options are compliance or total loss.

### 2.3 Why “just hide the app” is not enough

Renaming the icon or using a fake splash is insufficient if the full interaction path (notifications, Face ID sheet, balance screen, send flow) still looks like a wallet. Masking must survive the **full interaction**, not only the home-screen glyph. Design systems (including mobile UX workstreams such as **Cast**-style visual polish for consumer apps) should treat disguise as an end-to-end state machine, not a skin.

---

## 3. Core concept: duress password → decoy vault

| Element | Behavior |
|--------|----------|
| Identity | Same username / account handle |
| Primary password | Opens the **real vault** |
| Duress password | Opens a **decoy vault** holding a substantial but non-devastating sum (illustrative: **~$3,000**) |
| Session | Logout + primary password restores the real vault |
| Attacker view | Plausible balances, history, and spend UX inside the decoy |

### 3.1 Design goals for the decoy

- Believable under short inspection (recent activity, a few contacts, normal UI chrome).  
- Painful enough to “pay” under duress without being life-destroying.  
- Isolated cryptographically: signing keys for the decoy must not unlock real vault keys.  
- No accidental leakage of real balances via notifications, widgets, or clipboard.

### 3.2 Calibration note

~$3,000 is an **example**, not a fixed product constant. Calibration must consider perceived wealth, local cost of living, and typical wallet balances. See Open risks.

---

## 4. Biometric gating

**Claim:** Face ID (or equivalent) alone is insufficient outside a trusted context.

Proposed rule:

- **Inside home geofence:** biometric may unlock the last-used vault *or* present a password chooser, depending on product risk appetite.  
- **Outside home geofence:** biometric is only a device gate; a **password** is required, and that password selects **which vault** opens (real vs duress).

Rationale: coercion often happens away from home (street, vehicle, hotel). Forcing a second factor that encodes vault choice raises the cost of a successful “face only” attack without requiring the victim to invent a new account on the spot.

---

## 5. Geofence policy

Outside a configured radius (home / trusted places):

1. **Secondary password required** (vault selection).  
2. **Transfers above a threshold** (illustrative: **$1,000**) trigger:  
   - a **24-hour time-lock**, and/or  
   - **multi-sig approval** from a pre-enrolled trusted contact.

Inside the geofence, products may relax time-locks for ordinary spend (e.g. card rails like **RedotPay**) while still offering optional duress passwords.

Implementations should treat geofence as a **policy object**, not a hard-coded map, so partners can define multiple trusted zones.

---

## 6. App masking

- Present the wallet as a mundane app (e.g. gym tracker, notes, weather) until an unlock code / gesture reveals the real UI.  
- Masking must cover: icon, name, notification copy, Face ID prompt text, and deep links.  
- Failure mode to avoid: disguise that collapses on first biometric sheet or first push notification.

This is a **UX/feature** concern as much as a crypto concern; visual systems (e.g. **Cast**-quality consumer UI) should own the disguise state as a first-class mode.

---

## 7. Positioning: feature layer, not one rigid standard

Uniformity is predictable. If every wallet uses the same decoy amount, same delay, and same prompt copy, attackers learn the script.

**Propose instead:**

- An **adoptable feature layer** wallets and neobanks can implement partially.  
- A **minimal common vocabulary** so EtherFi-class wallets, RedotPay-class spend rails, and design partners can integrate without one frozen UI:

| Term | Meaning |
|------|---------|
| **Duress password** | Credential that opens the decoy vault for the same identity |
| **Geofence policy** | Trusted locations + rules when outside them |
| **Transfer threshold** | Amount above which cool-down / multi-sig applies |
| **Multi-sig hook** | Interface to request approval from a trusted contact / co-signer |

Optional profiles (aggressive / balanced / light) can share vocabulary while differing in defaults.

---

## 8. Open risks

| Risk | Notes |
|------|------|
| **GPS spoofing** | Attacker fakes “at home” to skip outside-fence rules; mitigate with OS location integrity, Wi-Fi/BLE corroboration, or fail-closed outside. |
| **Coercion at home** | Geofence does not help; rely on duress password + believable decoy. |
| **Decoy-amount calibration** | Too small → attacker demands the “real” password; too large → real loss under duress. |
| **Recovery after duress use** | Need safe re-key / rotate after a decoy session without tipping an observer still present. |
| **“Show me the real password”** | After a decoy transfer, attacker may escalate; training and plausible deniability matter; no crypto silver bullet. |
| **AML / KYC perception** | Duress and masking can look like evasion tooling; partners need clear lawful-use framing (personal safety), distinct from mixer semantics historically associated with **Tornado Cash**. |
| **Multi-sig key custody** | Trusted-contact keys create new custody and social-engineering surfaces. |
| **Notification / metadata leaks** | Widgets, email, and chain analytics can betray the real vault. |

---


---

## 9. Decoy Balance Calibration

Suggested decoy ranges are **illustrative defaults**, not compliance guidance. Inputs for tiering: on-chain history, public persona, social media visibility, and account age. Prefer under-matching a sophisticated attacker over over-funding a decoy.

| Perceived-wealth tier | Signals (examples) | Suggested decoy range (USD) | Rationale | When **not** to use this decoy |
|----------------------|--------------------|-----------------------------|-----------|--------------------------------|
| **Quiet / new user** | Thin on-chain history; new account; little or no public crypto persona | **$500 – $2,000** | Enough to look like a real hot wallet; loss is survivable | Attacker already saw a larger exchange KYC balance, payroll screenshot, or NFT floor that contradicts a tiny vault |
| **Moderate visible activity** | Months–years of ordinary txs; occasional social mentions; mid-size DeFi / card spend (EtherFi- / RedotPay-class patterns) | **$2,000 – $8,000** (default ~**$3,000**) | Matches “serious but not whale” expectation under short inspection | Attacker quotes an exact portfolio (tax form, shared screen, prior shoulder-surf); decoy would be obviously wrong |
| **High-profile / large history** | Public figure, known large bags, long / high-volume on-chain trail | **$15,000 – $50,000** (cap as % of real hot funds, e.g. ≤5–10%) | Low decoy fails immediately against known wealth; still not treasury | Attacker is informed (ex-partner, coworker, prior hack) and can verify exact holdings on-chain or via leaked exports — decoy mode may be worse than refusal / empty cold story |

### Dynamic vs static decoy funding

- **Default: static (or rarely reviewed) decoy.** Auto-chasing recent activity creates timing tells (sudden top-ups before travel) and can over-fund under a short coercion window.
- **Optional slow drift:** allow scheduled, capped rebalancing (e.g. monthly, ±10–20% band) tied to tier policy—not to every inbound payment.
- **Never auto-mirror** real vault balance or last N days of deposits dollar-for-dollar; that collapses plausible deniability if the attacker already sampled chain activity.


---

## 10. Multi-sig approver models (safety vs ease)

Scale: **Safety 1–5** (higher = harder for a coerced user alone to move large funds) · **Ease 1–5** (higher = less friction in daily life). Ratings are relative brainstorm scores, not audits.

| Model | How it works | Safety | Ease | Failure modes |
|-------|--------------|:------:|:----:|---------------|
| **(a) Trusted person, geographically separated** | Pre-enrolled human co-signer; policy requires min distance from user (e.g. different city) before their approval counts for out-of-fence large transfers | **5** | **2** | Approver unavailable / asleep; same-city travel breaks distance rule; social coercion of the approver; relationship change |
| **(b) Third-party service / custodian** | Policy engine or regulated co-signer API holds a policy key; auto or human-reviewed approve under SLA | **4** | **4** | Vendor outage; account takeover of the service; AML friction; trust concentration; cost |
| **(c) Hybrid — person primary, service fallback** | Ping trusted person first; if no response within N hours, escalate to service under the same threshold rules | **5** | **3** | Double-dependency outages; user confusion which path fired; fallback may be slower under active duress |
| **(d) Rotating approvers from a list** | Round-robin or random pick from a pre-approved set (family + friend + counsel) so one compromised contact is not enough | **4** | **2** | Coordination load; stale contacts; one weak link still approves; notification fatigue |
| **(e) Time-delayed self-approval** | No second party: above-threshold sends enter a long lock (e.g. 24–72h); user can cancel; after delay, self-finalize | **3** | **5** | Coercion lasting longer than the lock; attacker waits it out; no external check against “real password” escalation |

**Ranked shortlist (brainstorm):**  
1. **(c) Hybrid** — best safety/availability trade for partners.  
2. **(a) Separated person** — highest pure safety if relationships are healthy.  
3. **(b) Service** — easiest ops at scale.  
4. **(d) Rotating list** — strong for high-profile users, heavy UX.  
5. **(e) Delayed self** — easiest onboarding; weakest against patient attackers.

```
Safety █████  (a) person + distance
        ████░  (c) hybrid
        ████░  (b) service / (d) rotating
        ███░░  (e) delayed self

Ease    █████  (e) delayed self
        ████░  (b) service
        ███░░  (c) hybrid
        ██░░░  (a)/(d) human-heavy
```

---

## 11. Onboarding flow (screens)

Visual path (left → right). Each box is one screen.

```
[1 Welcome] → [2 Geofence] → [3 Decoy tier] → [4 Duress PW]
      → [5 Approver model] → [6 Transfer threshold] → [7 App masking]
      → [8 Simulation] → [9 Done]
```

| # | Screen | Shows | User inputs | Education copy (short) |
|---|--------|-------|-------------|------------------------|
| **1** | Welcome / education | Plain-language why banks have limits and wallets often don’t; coercion as a first-class threat | Continue / Not now | “This adds a decoy vault and slow-lane for big sends. It does not hide you from the chain.” |
| **2** | Home geofence | Map + radius slider; optional second trusted place | Pin home; set radius (e.g. 200m–2km) | “Outside this zone, Face ID alone won’t open your real vault.” |
| **3** | Decoy balance tier | Quiet / Moderate / High-profile cards + suggested USD ranges (§9) | Pick tier; optional custom within band | “Pick what looks believable for *you*. Too small can backfire.” |
| **4** | Duress password | Two fields + strength meter; never show side-by-side with primary PW | Set / confirm duress PW | “Same username. This password opens only the decoy. Practice it.” |
| **5** | Approver model | Cards for (a)–(e) with Safety/Ease chips | Choose one; enroll contacts or service | “Who (or what) must bless large sends when you’re away from home?” |
| **6** | Transfer threshold | Amount slider + currency; preview of time-lock duration | Set threshold (e.g. $1,000) + lock length | “Above this, outside the fence: delay and/or approval.” |
| **7** | App masking | Preview of mundane shell (e.g. gym tracker) vs real UI | Enable/disable; set reveal code / gesture | “Masking must survive notifications and Face ID sheets—not just the icon.” |
| **8** | Simulation mode | Safe sandbox: fake “outside fence + coercion” walkthrough; no real funds move | Run scenario; pass/fail checklist | “Practice once. Say the wrong password on purpose. See the decoy. Cancel freely.” |
| **9** | Done | Summary of choices; link to settings; optional trusted-contact invite | Finish / Invite approver | “You can change these later. After a real duress use, rotate passwords from a safe place.” |

**Simulation mode (detail):** scripted steps — (1) pretend GPS = outside, (2) Face ID OK, (3) enter duress PW → decoy balances, (4) attempt >threshold send → see time-lock / approver toast, (5) logout → primary PW → real vault. Badge every screen **SIMULATION · NO FUNDS MOVE**.

---

## 12. In-practice conditional flows (if-this → then-that)

Branching notation: `IF condition → UI state → NEXT`.

### (a) At home · normal login → real vault
```
IF inside geofence AND biometric OK AND (no PW required OR primary PW)
  → UI: Real vault home (true balances)
  → NEXT: normal send / card spend
STUCK RISK: user forgets they enabled “always ask PW even at home”
```

### (b) Outside geofence · Face ID · primary password → real vault + large-send brakes
```
IF outside geofence AND biometric OK
  → UI: Secondary password sheet (no vault hint)
IF password == primary
  → UI: Real vault
  → NEXT: sends ≥ threshold enter time-lock and/or approver queue
STUCK RISK: user thinks Face ID failed; copy must say “password required away from home”
```

### (c) Outside geofence · Face ID · duress password → decoy vault
```
IF outside geofence AND biometric OK AND password == duress
  → UI: Decoy vault (calibrated balance, plausible history)
  → NEXT: sends look instant in-decoy; no real-vault time-lock UI (avoid tell)
STUCK RISK: notifications from real vault leaking; suppress or rewrite while decoy session active
```

### (d) Transfer above threshold · outside geofence
```
IF real vault AND outside fence AND amount ≥ threshold
  → UI: “Pending · unlocks in 24h” + Approver notified
  → NEXT: wait | cancel | (if model allows) speed-up via second factor
STUCK RISK: user in café needs rent now — show clear cancel + lower-threshold settings path
```

### (e) Approver unreachable → fallback
```
IF approval pending AND approver timeout
  → UI: Fallback banner
  IF model == hybrid → escalate to service
  IF model == person-only → extend lock + “try another enrolled contact” (if rotating)
  IF model == delayed-self → continue countdown only
STUCK RISK: silent failure; always show countdown + who was pinged (without revealing decoy existence)
```

### (f) Panic-drain trigger
```
IF panic gesture/code entered (optional feature)
  → UI: brief confirm (or zero-confirm if configured for true emergency)
  → NEXT: move real hot funds to pre-set cold/deep path; decoy session may continue for cover
STUCK RISK: mistype under stress; require rehearsal in Simulation; irreversible without recovery ceremony
```

### (g) App masking active · wrong reveal code
```
IF masking ON AND launch/reveal code wrong
  → UI: Mundane app (gym tracker / notes) full interaction
  → NEXT: no wallet chrome, no crypto notifications
STUCK RISK: user forgets reveal code — recovery must not be a visible “Forgot wallet password?” on the decoy shell
```

### Quick decision tree
```
Launch
 ├─ masking ON & code wrong ──────────────→ Mundane UI (g)
 └─ masking OFF or code OK
      ├─ inside geofence ─────────────────→ Real vault (a) [policy may still ask PW]
      └─ outside geofence
           └─ Face ID → password sheet
                ├─ primary PW ────────────→ Real vault + threshold brakes (b)(d)(e)
                ├─ duress PW ─────────────→ Decoy vault (c)
                └─ panic code ────────────→ Panic-drain (f) then optional decoy cover
```

## 13. Phased next steps

1. **Spec + threat model** — formalize vault separation, geofence policy object, threshold / time-lock semantics, and coercion scenarios (home vs away).  
2. **Reference-wallet prototype** — one open reference implementing duress password, decoy vault, geofence gate, and masked UI shell.  
3. **Neobank / card pilot** — partner with a spend rail (pattern: **RedotPay**) and/or a self-custody product (pattern: **EtherFi**) to test UX under realistic send flows.  
4. **Publish vocabulary** — release the minimal terms (duress password, geofence policy, transfer threshold, multi-sig hook) for adopters; keep UI and defaults plural.

---

## 14. Non-goals

- Replacing hardware security modules or seed backup best practices.  
- Endorsing mixers or anonymity pools; **Tornado Cash** appears only as a historical example of post-theft irreversibility.  
- Mandating identical decoy amounts or identical UI chrome across vendors.

---

## 15. Closing

Duress-aware wallet security is a **product and policy layer** on top of self-custody: same identity, second password, decoy vault, geofenced high-value controls, and full-interaction app masking. Adopt it as a shared vocabulary and optional feature kit—so EtherFi-class wallets, RedotPay-class rails, and design partners (including **Cast**-grade UX) can harden coercion resistance without freezing into one attackable monoculture.

---

*Editable working draft · for engineers and partners · examples limited to EtherFi, RedotPay, Cast, Tornado Cash*
