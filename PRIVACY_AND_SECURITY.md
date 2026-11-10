# Privacy, Security & Runs on Atlassian Architecture

**Updated:** 10 November 2026  
**Vendor:** EvidencePair LLC  
**App:** Attachment Checkout for Confluence (`ari:cloud:ecosystem::app/e9b02536-a6a6-4d29-bb8f-4491b0aeb6f7`)

The app's privacy policy is [PRIVACY_POLICY.md](./PRIVACY_POLICY.md), and its security documentation is [SECURITY.md](./SECURITY.md). This page is a technical summary of both.

---

## 1. Runs on Atlassian, no data egress

Attachment Checkout for Confluence is a native Atlassian Forge application (`nodejs24.x` runtime):
- **No external remotes:** The app's `manifest.yml` declares **no** `remotes:` and **no** `permissions.external` (`fetch`, `frames`, `images`, `scripts`, or `styles`).
- **No third parties:** The app sends no customer content, attachment files, page titles, or user identifiers outside Atlassian's platform.
- **Where data is stored:** Checkout records, space settings and the hash-chained audit log are stored in **Atlassian Forge SQL** and **Atlassian Forge storage (KVS)**, which Atlassian hosts for the app. The app also writes a lock count (a page content property), Enforced-mode page restriction entries, and notification comments to the customer's Confluence site.

---

## 2. Data stored and processed

Attachment Checkout never reads or stores attachment file contents. It stores the metadata it needs to coordinate attachment locks and keep an audit trail:

| Data element | Where | Purpose |
|---|---|---|
| Atlassian `accountId`: lock `holder` and `released_by` | Forge SQL (`aco_locks`) | Who checked out, checked in or force-released an attachment. `erased-account` once the account is closed. |
| Atlassian `accountId`: event `actor` | Forge SQL (`aco_events`) | Who uploaded a new version during a checkout, or trashed or deleted a checked-out attachment. |
| Atlassian `accountId`: `updated_by` | Forge SQL (`aco_space_settings`) | The admin who last saved a space's settings. |
| Account IDs and group IDs in a page's edit restriction | Forge SQL (`aco_page_restrictions`) | In Enforced mode, the restriction before the first checkout, and the entries the app added. Deleted once the app has removed its entries. |
| Audit records: pseudonymous actor, action, attachment and page IDs, details (from this version, the holder and an uploader by pseudonym; records from earlier versions hold their `accountId`), SHA-256 chain hash | Forge SQL (`audit_events`) | Audit trail of check-outs, check-ins, force releases, expiries, concurrent uploads and closures. |
| Pseudonym to `accountId` mapping | Forge SQL (`audit_actors`) | Maps each audit pseudonym to the account it stands for. No display names. Blanked when the account is closed. |
| Each stored `accountId`, with when it was first tracked or last updated, last reported and reported closed | Forge SQL (`aco_personal_data_accounts`) | Personal data reporting. Deleted once a closed account is erased. |
| Space and content identifiers (`spaceKey`, `spaceId`, `pageId`, `attachmentId`) | Forge SQL | Associates locks with Confluence spaces, pages and attachments. |
| Optional user text: checkout `note`, check-in `comment`, force-release `reason` | Forge SQL (`aco_locks` and `audit_events`); the reason is also posted in the force-release notice | Shows checkout context to collaborators and records governance reasons. |
| Lock count (`aco-lock-summary`: byline title and tooltip) | Confluence page content property | Renders the "🔒 N checked out" byline. Holds no account IDs. |
| Notices with @mentions | Comments on the space's "Checkout notifications" Confluence page | Concurrent uploads, pre-expiry reminders, expiries and force releases. |
| Page coordination hold (`core-lock-v2`) | Forge KVS (60-second hold, renewed while a change runs, deleted when it ends) | Serializes concurrent checkout and check-in changes on the same page. |
| Audit pseudonym key, newest-record marker | Forge KVS (the key is a Forge storage secret) | Makes audit pseudonyms; anchors the audit hash chain. |
| Reporting cycle in seconds | Forge KVS | The cycle Atlassian last named for personal data reports. |

---

## 3. Personal data, retention and account closure (`report:personal-data`)

- **Reporting.** The hourly scheduled trigger (`aco-hourly`) reports the stored account IDs with Atlassian's personal data reporting API after its expiry sweep. It runs in its own `try`, so a reporting failure never stops the sweep, and a sweep failure never stops the report.
  - Each account is reported once per cycle. The cycle is Atlassian's `Cycle-Period`, or 7 days if Atlassian names none.
  - A request holds at most 90 accounts. A run makes at most 20 requests, and only while the invocation has time left. Accounts not reached wait for the next run, oldest first.
  - Each account is sent with its own `updatedAt`. The app's own account is never sent.
- **Closed accounts.** The app erases what it holds for an account Atlassian reports closed:
  - Active checkouts are released as an expiry would release them, with no notice, and recorded as `system:account-closed` and `erased-account`.
  - The ID becomes `erased-account` in `aco_locks`, `aco_events` and `aco_space_settings`.
  - The ID is removed from `aco_page_restrictions`. If an entry the app added for that person is still on the page, the app removes it first.
  - The `audit_actors` row is blanked.
  - Its `aco_personal_data_accounts` row is deleted.
  - Until every step has finished, the account stays marked closed, so a run that stops part-way is picked up an hour later.
- **Updated accounts.** The app stores no profile data, so it only moves that account's `updatedAt` forward.
- **Not erased.**
  - Comments the app posted on a "Checkout notifications" page. They are Confluence content.
  - A restriction entry Confluence refuses to remove.
  - Free text people typed.
  - Audit records written by earlier versions, which can hold a holder's or an uploader's account ID in their details. The app never changes audit records, so the hash chain stays verifiable. Records written from this version name those people by pseudonym.
- **Logs.** The report logs counts only. When it fails, the error is logged with the account IDs it was working on, and anything shaped like an account ID, replaced by `[account]`.
- **Pseudonyms.** Audit records name their actor by an HMAC-SHA256 pseudonym of the account ID, made with a random key the app creates once per installation (`pseudonymSecret`). The `audit_actors` table maps each pseudonym to its account ID, so while an account is open the pseudonym does not hide who acted.
- **Retention.** Checkouts, checkout events, space settings, the audit log and the actor directory are kept with no time limit. The app has no retention setting and no function to delete them.
- **Uninstalling.** The app has no uninstall handler, so it deletes nothing when it is uninstalled. What then happens to its Forge SQL and KVS data is decided by Atlassian's platform.

The privacy policy gives the full list of fields and how long each is kept.

---

## 4. Confluence permission scopes and permission checks

Every user-invoked Forge resolver that uses an app-level (`asApp()`) client first checks the caller's Confluence permissions **as the calling user (`asUser()`)** (`CORE-AC-03`). The page checks use `POST /wiki/rest/api/content/{id}/permission/check`, which needs the scope `read:confluence-content.permission` (added in version 4.0.0). Space-admin checks read the caller's permitted operations from Confluence's v2 space API (`GET /wiki/api/v2/spaces/{id}/operations`). [SECURITY.md](./SECURITY.md) lists every scope in the manifest and what it is used for.

| Operation | Required caller permission guard |
|---|---|
| `getModalState`, `getAttachmentHistory` | `requirePageViewer` (`POST /wiki/rest/api/content/{id}/permission/check` with `operation: 'read'`) |
| `checkOut` | `requirePageEditor` (`POST /wiki/rest/api/content/{id}/permission/check` with `operation: 'update'`) |
| `checkIn` | `requirePageViewer` on the lock's page, then the caller must be the lock holder (`lock.holder === caller.accountId`) |
| `forceRelease` | `requireConfluenceAdmin`, `requireSpaceAdmin` (v2 space operations) on the lock's space, or membership in the space's `forceReleaseGroupIds` |
| `getSpaceSettings`, `saveSpaceSettings` | `requireConfluenceAdmin` or `requireSpaceAdmin` (v2 space operations) |
| `getOverview`, `exportOverviewCsv` | `requireConfluenceAdmin` (or `mineOnly: true` scoped strictly to the signed-in caller's `accountId`) |
| Background triggers (`onEvent`, `onHourly`, `onMaintenance`) | `requireSystemActor` (closed-list system contexts: `'product-event'` for `onEvent`, `'scheduled-trigger'` for `onHourly`, including its personal data report, and `onMaintenance`) |
| Install and upgrade (`onLifecycle`) | Runs the database migrations only; makes no Confluence requests |

`getModalState` and `getAttachmentHistory` check the page named in the request. This version does not check that the requested attachment, or the space whose settings `getModalState` returns, belongs to that page.

---

## 5. Security contact and vulnerability disclosure

To report a security question or vulnerability, email EvidencePair LLC at security@evidencepair.com. We acknowledge every report within one business day. For product support, see <https://evidencepair.com/support/>.
