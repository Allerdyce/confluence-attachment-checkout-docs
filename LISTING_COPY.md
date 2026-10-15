# Marketplace Listing Copy & Version Highlights — Attachment Checkout for Confluence

## App Identity
- **Name:** Attachment Checkout for Confluence
- **Tagline (Short Description):** Native Forge attachment check-out/check-in locks with auto-expiry, @mentions, space governance & CSV audit export.
- **Category:** Document Management / Compliance & Governance
- **Runs on Atlassian:** Yes (100% Native Forge, Zero Remotes, Zero Data Egress)
- **Vendor:** EvidencePair LLC

---

## Detailed Overview (Marketplace Description)

### The Enterprise Native-Forge Replacement for Legacy Connect Lock Apps
Atlassian Connect reaches official **End of Support on January 31, 2027**. Enterprise Confluence teams migrating off legacy Connect attachment-locking tools (such as Cenote Lockpoint) or outgrowing basic Forge lock utilities need a **100% Native Forge, Runs on Atlassian (Zero Remotes / Zero Data Egress)** solution built for real document governance.

**Attachment Checkout for Confluence** prevents teammates from overwriting binary attachments (Word, Excel, PowerPoint, PDF, Visio, and CAD files) while keeping 100% of your lock state and audit history inside Atlassian-hosted Forge SQL.

### Four Enterprise Differentiators
1. **Automated Lock Expiry:** Configure default (3-day) and maximum (up to 365-day) lock durations per space so vacationing or departing employees never permanently block page attachments.
2. **Real-Time `@Mention` Notifications:** Automatically posts Confluence `@mention` notifications on a dedicated per-space **"Checkout notifications"** page for concurrent-upload violations, 24-hour pre-expiry warnings, auto-expirations, and admin force-releases—without cluttering content pages or sending data to external email servers.
3. **Space-Level Governance & Force-Release (`Advisory` & `Enforced` Modes):** Choose **Advisory** mode (status badges + concurrent-upload detection) or **Enforced** mode (page edit restrictions that block non-holder uploads and restore prior restrictions cleanly upon check-in). Space admins and delegated groups can force-unlock any file with a mandatory audit reason.
4. **Compliance CSV Audit Export:** 1-click UTF-8 CSV export of active locks and checkout metadata for ISO 27001, SOC 2, and GxP document control audits.

---

## Version Highlights (Highlight Layout — 3 Mandatory Highlights)

### Highlight 1: One-Click Attachment Check-Out, Check-In & Enforced Page Locks
- **Title:** Prevent Conflicting Edits with Advisory or Enforced Attachment Locks
- **Description:** Check out individual files or batch-select attachments right from the page byline (`🔒 N checked out`) or content action menu. Add collaborator notes, track exact attachment versions at checkout and check-in, and view full per-attachment lock and violation history.
- **Caption:** Page Attachment Checkout Modal showing active locks, version tracking, concurrent-upload warnings, and attachment history.
- **Screenshot File:** `screenshots/01-page-attachment-checkout-modal.png` (1440×900)

### Highlight 2: Automated Lock Expiry, Space Governance & `@Mention` Alerts
- **Title:** Never Get Stuck Behind a Vacationing Teammate's Stale Lock
- **Description:** Space admins configure Advisory vs. Enforced mode, default and maximum auto-expiry durations (1–365 days), admin groups, and delegated force-release groups. Automated hourly sweeps send 24-hour pre-expiry `@mention` notices and expire overdue locks automatically.
- **Caption:** Space Settings governance panel configuring Advisory/Enforced mode, auto-expiry limits, and force-release groups.
- **Screenshot File:** `screenshots/02-space-settings-expiry-guards.png` (1440×900)

### Highlight 3: Global Active-Lock Overview & 1-Click Compliance CSV Audit Export
- **Title:** Site-Wide Lock Visibility, Force-Release & ISO 27001 / SOC 2 CSV Exports
- **Description:** Confluence admins search active checkouts across every space, filter by holder or mode, force-release locks with a required audit reason, and export a formula-safe UTF-8 CSV audit snapshot in one click. Every user also gets a personal "My checkouts" queue.
- **Caption:** Global Active Locks Overview with space/holder/mode filters, admin force-release, and 1-click CSV audit export.
- **Screenshot File:** `screenshots/03-global-lock-overview-csv-export.png` (1440×900)
