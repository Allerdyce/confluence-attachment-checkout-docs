# Privacy, Security & Runs on Atlassian Architecture

**Effective Date:** October 14, 2026  
**Vendor:** EvidencePair LLC  
**App:** Attachment Checkout for Confluence (`ari:cloud:ecosystem::app/e9b02536-a6a6-4d29-bb8f-4491b0aeb6f7`)

---

## 1. 100% Runs on Atlassian — Zero Data Egress

Attachment Checkout for Confluence is a **100% Native Atlassian Forge application** (`nodejs24.x` runtime):
- **Zero External Remotes:** The app's `manifest.yml` declares **no** `remotes:` and **no** `permissions.external` (`fetch`, `frames`, `images`, `scripts`, or `styles`).
- **Zero Third-Party Subprocessors:** No customer content, attachment binaries, page titles, or user identifiers ever leave Atlassian's Cloud infrastructure.
- **In-Realm Storage:** All checkout records, space settings, and tamper-evident SHA-256 hash-chained audit logs are stored exclusively inside **Atlassian Forge SQL** and **Atlassian Forge Hosted Storage (KVS)** within the customer's Atlassian Cloud tenant.

---

## 2. Data Stored & Processed

Attachment Checkout never reads or stores attachment file bodies. It stores only the minimal metadata required to coordinate attachment locks and maintain an audit trail:

| Data Element | Storage Mechanism | Purpose |
|---|---|---|
| Atlassian `accountId` (`holder`, `releasedBy`, `actor`) | Forge SQL (`aco_locks`, `aco_lock_events`, `core_audit_log`, `core_actors`) | Identifies who checked out, released, or modified a locked attachment. |
| Space & Content Identifiers (`spaceKey`, `spaceId`, `pageId`, `attachmentId`) | Forge SQL & Page Content Property (`aco-lock-summary`) | Associates locks with Confluence pages and attachments. |
| Optional User Text (`note`, check-in `comment`, force-release `reason`) | Forge SQL | Displays checkout context to collaborators and records governance reasons. |
| Short-Lived Page Coordination Lock (`core-lock-v2`) | Forge KVS (60-second TTL) | Serializes concurrent checkout/check-in mutations on the same page. |

---

## 3. Personal Data Lifecycle & GDPR Erasure (`report:personal-data`)

Attachment Checkout implements Atlassian's Forge Personal Data Reporting API (`report:personal-data`):
- Every Atlassian `accountId` stored in the app's audit and actor tables is periodically reported to Atlassian's `/app/report-accounts` lifecycle endpoint.
- When Atlassian notifies the app that an account has been closed (`status: 'closed'`), the app pseudonymizes that user's identifier using an installation-scoped HMAC-SHA256 key (`pseudonymSecret`) and purges personal references in accordance with GDPR / CCPA right-to-erasure requirements while preserving audit chain integrity.
- When a customer uninstalls the app, Atlassian's platform lifecycle removes the tenant's Forge SQL and KVS partitions according to Atlassian's standard Forge data-retention schedule.

---

## 4. Confluence Permission Scopes & Defense-in-Depth Guards

Every user-invoked Forge resolver verifies the caller's Confluence permissions **as the calling user (`asUser()`)** before any app-level (`asApp()`) client is constructed (`CORE-AC-03`):

| Operation | Required Caller Permission Guard |
|---|---|
| `getModalState`, `getAttachmentHistory` | `requirePageViewer` (`POST /wiki/rest/api/content/{id}/permission/check` with `operation: 'read'`) |
| `checkOut` | `requirePageEditor` (`POST /wiki/rest/api/content/{id}/permission/check` with `operation: 'update'`) |
| `checkIn` | Lock holder (`lock.holder === caller.accountId`) or authorized force-releaser |
| `forceRelease` | `requireConfluenceAdmin`, `requireSpaceAdmin`, or membership in the space's `forceReleaseGroupIds` |
| `getSpaceSettings`, `saveSpaceSettings` | `requireConfluenceAdmin` or `requireSpaceAdmin` |
| `getOverview`, `exportOverviewCsv` | `requireConfluenceAdmin` (or `mineOnly: true` scoped strictly to the signed-in caller's `accountId`) |
| Background Triggers (`onEvent`, `onHourly`, `onLifecycle`) | `requireSystemActor` (closed-list system contexts `'product-event'` and `'scheduled-trigger'`) |

---

## 5. Security Contact & Vulnerability Disclosure

To report a security question or vulnerability, contact EvidencePair LLC at `support@titlecovenant.com`. Reports are acknowledged within 24 hours.
