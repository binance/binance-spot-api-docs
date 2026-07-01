# CHANGELOG for Binance SPOT Demo Mode

**Last Updated: 2026-07-01**

### 2026-07-01

* Scheduled downtime maintenance will begin at **2026-07-03 07:00 UTC** and will last for approximately 4 hours.
* This window will include upgrades to our infrastructure.

---

### 2026-05-11

The following rollout will occur at **approximately 07:00 UTC on 2026-05-12**.

* Added WebSocket Stream support for [Block Trades](https://www.binance.info/en/support/faq/detail/557f95eaf8fb4460aed0a891d42a1425).
  * New stream:
    * `<symbol>@blockTrade`

---

### 2026-03-12

**Notice:** FIX TLS Connectivity Update on **2026-06-08**, starting from **03:00 UTC** and will take about 1 hour to complete.

**Action Required:**

During the update window, existing FIX connections may drop intermittently. To ensure successful reconnections and new connections afterward, please verify before our update that your client sends SNI (Server Name Indication) during the TLS handshake and validates the certificate against the requested hostname. <br>
Clients without SNI may receive an error message during handshake related to incorrect certificate during or after the update window, leading to TLS handshake or hostname verification failures. This can occur with some Node.js clients if SNI is not explicitly configured.<br>
Please consult the [FIX API documentation](../fix-api.md#general-api-information) for full context.

---

### 2026-03-09

Scheduled downtime maintenance will begin at **2026-03-13 06:00 UTC** and will last for approximately 4 hours.

---

### 2026-01-29

This changelog will announce any scheduled downtime for maintenance for the SPOT Demo Mode environment.

For more information on how to use Demo Mode via API, please refer to the [General Info](general-info.md) page.
