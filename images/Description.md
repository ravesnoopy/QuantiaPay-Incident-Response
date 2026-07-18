# Images
### Incident IR-2026-0847 · QuantiaPay

> Screenshots captured from the IR Console during the live simulation.
> Replace each `image.png` placeholder with your actual screenshot.

---

## 01 — SIEM P1 Alert

> The correlation rule fires at 03:14:00 — anomalous service account activity combined with active external exfiltration.

![SIEM P1 Alert](siem-p1-alert.png)

---

## 02 — Netstat Output (rclone.exe)

> Live network connection showing `rclone.exe` (PID 4188) actively connected to `198.51.100.23:443` at time of capture.

![Netstat rclone](Netstat%20rclone.PNG)

---

## 03 — RAM Capture

> 16 GB memory image secured from `jump-02`. Contains `rclone.exe` in memory and `temp.conf` with the exfiltration destination.

![RAM Capture](Ram.PNG)

---

## 04 — Verify All (6/6 OK)

> All six post-eradication checkpoints confirmed clean.

![Verify All](verifyall.PNG)

---

## 05 — Final Score

> Session score: 99/100 across all five response phases.

![Final Score](FinalScore.PNG)

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
