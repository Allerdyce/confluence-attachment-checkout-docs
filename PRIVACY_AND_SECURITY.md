# Privacy, Security & Runs on Atlassian Architecture

**Updated:** 9 November 2026  
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
| Atlassian `accountId`: lock `holder` and `released_by` | Forge SQL (`aco_locks`) | Who checked out, checked in or force-released an attachment. |
| Atlassian `accountId`: event `actor` | Forge SQL (`aco_events`) | Who uploaded a new version during a checkout, or trashed or deleted a checked-out attachment. |
| Atlassian `accountId`: `updated_by` | Forge SQL (`aco_space_settings`) | The admin who last saved a space's settings. |
| Account IDs and group IDs in a page's edit restriction | Forge SQL (`aco_page_restrictions`) | In Enforced mode, the restriction before the first checkout, and the entries the app added. Deleted once the app has removed its entries. |
| Audit records: pseudonymous actor, action, attachment and page IDs, details (which can include the holder's and an uploader's `accountId`), SHA-256 chain hash | Forge SQL (`audit_events`) | Audit trail of check-outs, check-ins, force releases, expiries, concurrent uploads and closures. |
| Pseudonym to `accountId` mapping | Forge SQL (`audit_actors`) | Maps each audit pseudonym to the account it stands for. No display names. |
| Space and content identifiers (`spaceKey`, `spaceId`, `pageId`, `attachmentId`) | Forge SQL | Associates locks with Confluence spaces, pages and attachments. |
| Optional user text: checkout `note`, check-in `comment`, force-release `reason` | Forge SQL (`aco_locks` and `audit_events`); the reason is also posted in the force-release notice | Shows checkout context to collaborators and records governance reasons. |
| Lock count (`aco-lock-summary`: byline title and tooltip) | Confluence page content property | Renders the "🔒 N checked out" byline. Holds no account IDs. |
| Notices with @mentions | Comments on the space's "Checkout notifications" Confluence page | Concurrent uploads, pre-expiry reminders, expiries and force releases. |
| Page coordination hold (`core-lock-v2`) | Forge KVS (60-second hold, renewed while a change runs, deleted when it ends) | Serializes concurrent checkout and check-in changes on the same page. |
| Audit pseudonym key, newest-record marker | Forge KVS (the key is a Forge storage secret) | Makes audit pseudonyms; anchors the audit hash chain. |

---

## 3. Personal data, retention and account closure (`report:personal-data`)

- The manifest declares Atlassian's `report:personal-data` scope, but this version **does not call** Atlassian's personal data reporting API. It does not report the account IDs it stores, it never learns that an account has been closed, and it erases nothing when one is.
- Audit records name their actor by an HMAC-SHA256 pseudonym of the account ID, made with a random key the app creates once per installation (`pseudonymSecret`). The `audit_actors` table maps each pseudonym to its account ID, and some audit details hold account IDs, so the pseudonym does not hide who acted.
- Checkouts, checkout events, space settings, the audit log and the actor directory are kept with no time limit. The app has no retention setting and no function to delete them.
- The app has no uninstall handler, so it deletes nothing when it is uninstalled. What then happens to its Forge SQL and KVS data is decided by Atlassian's platform.

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
| Background triggers (`onEvent`, `onHourly`, `onMaintenance`) | `requireSystemActor` (closed-list system contexts: `'product-event'` for `onEvent`, `'scheduled-trigger'` for `onHourly` and `onMaintenance`) |
| Install and upgrade (`onLifecycle`) | Runs the database migrations only; makes no Confluence requests |

`getModalState` and `getAttachmentHistory` check the page named in the request. This version does not check that the requested attachment, or the space whose settings `getModalState` returns, belongs to that page.

---

## 5. Security contact and vulnerability disclosure

To report a security question or vulnerability, email EvidencePair LLC at security@evidencepair.com. We acknowledge every report within one business day. For product support, see <https://evidencepair.com/support/>.
