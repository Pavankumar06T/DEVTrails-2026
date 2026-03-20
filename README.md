# 🚚 KaavalPay — Parametric Insurance Platform for Gig Workers

- **Kaaval** (காவல்) means *Protection* in Tamil.
- Because every delivery partner deserves protection when the storm hits.
- KaavalPay is an AI-powered parametric insurance platform designed to protect delivery partners during adverse weather events. When a verified weather alert triggers in a partner's active delivery zone, eligible workers automatically receive a payout — no paperwork, no claims processing delay.

Built for the DEVTrails 2026 Hackathon | Guidewire Track

---

## 🔧 Core Platform Architecture

- **Trigger Engine:** Integrates with weather APIs (IMD, OpenWeatherMap) to detect red-alert zones in real time
- **Partner Registry:** Tracks active delivery partners with live location heartbeats
- **Payout Engine:** Automated IMPS/UPI disbursement when trigger + eligibility conditions are met
- **Dashboard:** Insurer-facing dashboard showing live zone alerts, claim volume, and fraud risk heatmaps

---

## 🛡️ Adversarial Defense & Anti-Spoofing Strategy

> *Added in response to the Market Crash threat scenario — coordinated GPS spoofing by organized fraud rings targeting parametric payout triggers.*

---

### 1. Differentiating Genuine Stranded Workers from GPS Spoofers

Simple GPS coordinates are a single point of truth — and a single point of failure. Our architecture replaces coordinate-trust with **multi-signal behavioral verification**, scoring each claim across five independent data streams:

**Signal Layer 1 — Device Sensor Fusion**

GPS can be faked. Accelerometers, gyroscopes, and barometers cannot be simultaneously faked without hardware-level spoofing. A genuinely stranded delivery partner will show:
- Accelerometer data consistent with being stationary in adverse conditions (micro-vibrations from wind/rain, not flat zero-motion of a spoofed device)
- Barometric pressure readings that match the claimed weather zone's atmospheric data from our ground-truth weather API
- If GPS says "Andheri West during a flood" but the barometer reads clear-sky pressure — it is flagged immediately

**Signal Layer 2 — Network & Connectivity Fingerprinting**

- A real worker in a flood zone will show **degraded, intermittent connectivity** — packet loss, signal tower switching, dropped pings. Our backend logs connection quality metadata on every heartbeat.
- A spoofed device sitting at home shows **stable 4G/WiFi** with clean latency. This contradiction is detectable.
- Cell tower triangulation is cross-referenced against the claimed GPS location. If the phone is pinging a tower 12km away from the claimed zone, it is an automatic risk flag.

**Signal Layer 3 — Historical Mobility Pattern Analysis**

Every registered delivery partner builds a 30-day mobility graph. Our anomaly detection model (Isolation Forest) flags:
- A partner who has never worked in Zone X suddenly "appearing" there during an alert
- Claiming to be stranded at 2 AM when their historical pattern shows they never work past 10 PM
- Zero delivery movement in the 4 hours before the stranded claim (a real worker would have movement history leading into the zone)

**Signal Layer 4 — App Integrity Verification**

At claim submission, our SDK performs a silent device integrity check (Google Play Integrity API / Apple DeviceCheck):
- Is the app running in an emulator?
- Is the device rooted or jailbroken?
- Is the GPS provider flagged as a mock location by the OS?

Android exposes `Location.isFromMockProvider()` — if true, the claim is auto-flagged before reaching the fraud scoring pipeline.

**Signal Layer 5 — Claim Velocity & Timing Correlation**

Genuine mass weather events produce geographically distributed claims over a rolling time window. A fraud ring produces a spike — hundreds of claims within minutes. Our system tracks:
- Claims-per-minute per zone vs. historical baseline
- Time delta between weather alert trigger and first claim submission (fraud rings monitor alerts and mass-submit immediately; real workers submit when actually stuck)

---

### 2. Data Points Used to Detect a Coordinated Fraud Ring

| Data Point | What It Reveals |
|---|---|
| Shared device fingerprints | Same device used by multiple partner IDs |
| Payout destination clustering | Multiple partners routing payouts to the same UPI/bank account |
| Claim submission IP addresses | Fraud rings often submit from the same network block |
| Inter-claim timing patterns | Suspiciously uniform time gaps suggest scripted submissions |
| Partner referral network graph | Ring members often joined the platform via the same referral chain |
| Weather API ground truth vs. claim zone | Is the claimed zone actually experiencing the alert-level event? |
| Historical claim frequency per partner | Zero prior claims → sudden high-value claim in a new zone |
| Cell tower vs. GPS discrepancy | Tower pings contradict the spoofed GPS coordinate |

Our system builds a **Fraud Ring Graph** — if Partner A and Partner B submit claims within 90 seconds from the same GPS zone and share a payout bank, and Partner B is connected to Partner C via referral, the entire cluster is flagged for manual review. This graph-based approach catches rings that individually look legitimate but collectively expose coordination.

---

### 3. UX Balance — Handling Flagged Claims Without Punishing Honest Workers

A delivery partner in a genuine flood with a laggy GPS and bad signal looks identical to a spoofer on raw data. Our framework uses a **3-tier response system** instead of binary approve/deny:

**Tier 1 — Auto-Approved (Risk Score < 30)**
- All signals consistent, no anomalies detected
- Payout processed immediately, zero friction for the worker

**Tier 2 — Soft Challenge (Risk Score 30–70)**
- Worker receives a one-tap prompt: *"We're verifying your location — share a 10-second video or photo of current conditions to speed up your payout."*
- Framed as help, not accusation
- Payout is held for 15 minutes during verification, not denied
- A genuinely stranded worker can easily share a clip. A fraudster at home cannot fake a flood.

**Tier 3 — Manual Review Queue (Risk Score > 70)**
- Flagged and routed to a human reviewer
- Worker receives: *"Your claim is under priority review. You will hear back within 2 hours. Your claim is NOT denied."*
- If cleared: payout is processed immediately with a small goodwill credit as acknowledgment of the delay
- If confirmed fraud: account suspended and added to the ring detection graph

**Appeals Layer**

Every denied claim gets a one-click appeal flow with full audit trail. False positives are tracked as a model performance KPI — if our false positive rate exceeds 5%, risk scoring thresholds are automatically recalibrated.

**Core Principle:** An honest worker always gets their money. The worst case is a 2-hour delay with clear, compassionate communication — never a silent denial.

---

### Architecture Flow
```
Claim Submitted
      │
      ▼
[Mock GPS Detection] ── flagged? ──→ Auto-Deny + Log
      │ clean
      ▼
[Device Integrity Check] ── flagged? ──→ Tier 3 Manual Review
      │ clean
      ▼
[Multi-Signal Risk Scorer]
  ├── Sensor Fusion Score
  ├── Network Quality Score
  ├── Mobility Anomaly Score
  └── Claim Velocity Score
      │
      ▼
  Score < 30  → Auto-Approve
  Score 30–70 → Soft Challenge (photo/video verification)
  Score > 70  → Manual Review Queue
      │
      ▼
[Fraud Ring Graph Builder]
  → Clusters connected suspicious actors
  → Triggers coordinated account suspension
```

---

## 👥 Team

- Pavankumar T
- Mahadev R
- Santhosh M
- Pradeepraja A
- Athuzhai N H

---

## 📌 Submission Notes

- Phase 1 submission for DEVTrails 2026
- Adversarial Defense section added in response to the Market Crash scenario
- No GPS coordinate alone is trusted — every claim is scored across 5 independent signal layers
