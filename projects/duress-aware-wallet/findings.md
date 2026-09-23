# Threat-model findings (companion to matrix)

**Diagram:** `duress-aware-threat-model-matrix.png`  
**Proposal:** `duress-aware-wallet-security-standard.md`

## Key findings

1. **Best coverage is away-from-home coercion with a live phone.** Armed robbery or kidnapping outside the geofence is where the stack works together: duress password → decoy vault, geofence-gated second password, time-lock and/or multi-sig on large sends, optional panic-drain.

2. **Forced biometric alone is not enough if vault choice is password-bound.** Outside the fence, Face ID should only unlock the device gate; which vault opens depends on the password. That breaks the “face = full treasury” failure mode.

3. **GPS spoofing punches a hole in geofence policy.** If “at home” can be faked, secondary-password and outside-fence rules may never fire. Multi-sig and time-locks still help; geofence itself does not. Treat location integrity as a hard dependency or fail closed.

4. **Largest product gap: attacker demands the “real” password after a decoy transfer.** No cell in that row is green. Calibration, training, and plausible deniability matter more than another control checkbox—and there is no cryptographic fix once the attacker knows a decoy exists.

5. **Seed-phrase social engineering is largely out of band.** App masking helps slightly (harder to spot a wallet). Duress/geofence/time-lock/multi-sig do not stop someone giving away a seed. Separate backup and recovery hygiene are required.

6. **Mixers make post-theft recovery fail; pre-broadcast controls are the only window.** Time-lock and multi-sig are the primary brakes before funds hit irreversible privacy rails. Duress/decoy only limits how much leaves in the first session.

7. **Home coercion (inside geofence) is an explicit gap in geofence-gated defenses.** Inside the radius, rely on duress password + believable decoy; do not assume location policy saves the user.

8. **Panic-drain is optional and partial.** Useful as a last-resort movement of real funds to a deep cold path, but dangerous if discoverable or mistyped under stress—keep it optional and well-rehearsed.
