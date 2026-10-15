# Attachment Checkout for Confluence — User & Administrator Guide

## 1. Overview

Attachment Checkout for Confluence prevents conflicting edits on binary attachments (Word documents, Excel spreadsheets, CAD drawings, PDFs, and presentations) attached to Confluence Cloud pages.

Every page displays a lightweight byline summary (**Ready for checkout** or **🔒 N checked out**) backed by a Confluence content property—requiring **zero function invocations on page view**—and provides a full **Attachment Checkout** modal from both the page byline and the page actions (`•••`) menu.

---

## 2. Checking Out & Checking In Attachments

### Checking Out a File
1. Open any Confluence page containing attachments.
2. Click **Ready for checkout** (in the page byline) or open the page actions menu (`•••`) and select **Attachment Checkout**.
3. For a single attachment, click **Check out** in the row's **Actions** column. For multiple attachments, select their checkboxes, optionally enter a **Checkout note** and **Duration (days)**, and click **Check out selected**.
4. While checked out:
   - The page byline updates to `🔒 N checked out` with a tooltip listing the holders and expiry timestamps.
   - Download and edit the attachment locally, then upload your new revision to the Confluence page.

### Checking In a File
1. Re-open the **Attachment Checkout** modal on the page.
2. Optionally enter a **Check-in comment** describing your changes.
3. Click **Check in** next to your locked attachment.
4. The lock is released, the new attachment version is recorded in the checkout history, and (in Enforced mode) the page's prior edit restrictions are restored.

---

## 3. Space-Level Governance (`Space Settings → Attachment Checkout`)

Space administrators and Confluence administrators can configure Attachment Checkout per space under **Space Settings → Apps → Attachment Checkout**:

| Setting | Default | Description |
|---|---|---|
| **Enable Attachment Checkout** | Enabled (`true`) | Enables checkout controls in the space and ensures the space's **Checkout notifications** page exists. |
| **Checkout mode** | `advisory` | Choose between `advisory` (warnings & violation tracking) and `enforced` (*"Locks editing of the whole page while any attachment is checked out."*). |
| **Default auto-expiry duration (days)** | `3` days | Default lock duration when a user does not specify one (`1–365` days). |
| **Maximum auto-expiry duration (days)** | `30` days | Upper bound on any checkout duration in the space (`defaultDurationDays ≤ maxDurationDays ≤ 365`). |
| **Admin group ID** | Site admins group | Group retained alongside the lock holder in Enforced mode so administrators are never locked out. |
| **Force-release group IDs** | Optional list | Additional Confluence group IDs authorized to force-release active locks in this space. |

---

## 4. Force-Releasing a Stale or Urgent Lock

If a lock holder is unavailable before auto-expiry:
1. A Confluence administrator, space administrator, or member of a configured **Force-release group** opens the **Attachment Checkout** modal (or the global **Attachment Checkout** overview page).
2. Click **Force release** on the locked attachment.
3. Enter a mandatory **Force release reason** for the tamper-evident audit trail and click **Confirm force release**.
4. The lock is immediately released and an `@mention` comment is posted on the space's **Checkout notifications** page alerting the previous holder with the actor and reason.

---

## 5. `@Mention` Notifications (`Checkout notifications` Page)

Attachment Checkout never spams content pages with comments and never sends data to external email relays. Instead, when enabled in a space, it creates a single dedicated page titled **Checkout notifications** and posts storage-format `@mention` footer comments for:
- **Concurrent Upload Violations:** When another user uploads a new revision of an attachment while it is checked out.
- **24-Hour Pre-Expiry Reminders:** Sent during the hourly maintenance sweep when an active lock is within 24 hours of expiring.
- **Automated Lock Expiry:** When a lock reaches its `expiresAt` timestamp and is automatically released.
- **Admin Force-Release:** When an administrator or delegated group member force-releases a lock.

---

## 6. Global Overview, Personal Queue & Compliance CSV Export

Open **Apps → Attachment Checkout** from the top Confluence navigation bar:
- **All active checkouts (Confluence Admins):** Filter active locks across the entire site by **Space key**, **Holder accountId**, or **Mode** (`advisory` / `enforced`).
- **My checkouts (All Signed-In Users):** View and check in all attachments you currently hold across every space.
- **Export CSV:** Click **Export CSV** to download a UTF-8 BOM CSV snapshot (`attachment-checkouts.csv`) containing `spaceKey`, `pageId`, `attachmentId`, `holder`, `mode`, `checkedOutAt`, `expiresAt`, `versionAtCheckout`, and `note`—ready for ISO 27001 / SOC 2 document-control evidence.
