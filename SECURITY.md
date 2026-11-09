# Security: Attachment Checkout for Confluence

Updated 9 November 2026

How the Atlassian Marketplace app **Attachment Checkout for Confluence**, made by EvidencePair LLC, handles data, what protects it, how we handle vulnerabilities, and which certifications apply. What the app stores is listed in full in its [privacy policy](./PRIVACY_POLICY.md).

## Data handling

- **Forge only.** The app runs on Atlassian Forge (`nodejs24.x`) on Atlassian infrastructure.
- **No egress.** The manifest declares no external permissions and no remote services. The app makes requests only to your site's Confluence APIs and its own Forge storage, and it loads no analytics, error-reporting services or external assets. The space settings screen links to the user guide on GitHub. That is a link a person can choose to open, and the app sends nothing there.
- **Where its data is.** The app keeps its records in Forge SQL and Forge storage, which Atlassian hosts for the app. It writes a lock count (the `aco-lock-summary` content property), Enforced-mode restriction entries and notification comments to your Confluence site. It sends none of this to EvidencePair.
- **A permission check comes first.** Every request from the app's screens first checks, as the person using it, that they may do what they ask:
  - open the app's window on a page, or an attachment's history: view the page;
  - check out: edit the page;
  - check in: view the page, and be the holder;
  - force-release: be a Confluence admin, an admin of the checkout's space, or a member of one of the space's force-release groups;
  - view or save space settings: be a Confluence admin or an admin of the space;
  - see every active checkout on the site, or export it: be a Confluence admin. "My checkouts" needs only a signed-in person and shows only their own.

  Only after the check passes does the app read and write Confluence as the app. Restriction entries, the content property, the notifications page and its comments are written by the app's own account. When Confluence refuses to let the app remove a restriction entry, the app tries the removal again as the person releasing the checkout. The app never acts as anyone other than itself or the person using it.
- **A gap in the page check.** The check covers the page named in the request. This version does not check that the attachment, or the space whose settings it returns, belongs to that page. Someone who can view any page can therefore read the checkout history of any attachment, and the settings of any space, if they know its ID or key.
- **Background work runs as the app.** The attachment and page event handlers and the hourly job run without a signed-in user. They act as the app to record concurrent uploads, close checkouts whose attachment or page was trashed or deleted, expire checkouts, post notices and update lock counts. The install and upgrade handler only sets up the app's database tables.
- **Permissions it asks for.**
  - `storage:app`: keep the app's records in Forge SQL and Forge storage.
  - `read:confluence-content.permission`: check whether the person can view or edit a page (`POST /wiki/rest/api/content/{id}/permission/check`, as the person). Opening the app's window, checking out, checking in and reading an attachment's history all run this check first. Version 4.0.0 added this scope.
  - `read:confluence-user`: read the app's own account ID, and whether the person is a Confluence admin.
  - `read:space:confluence`: find a space by its key, and read the person's permitted operations in it to check for space admins. Since version 4.0.0 this uses Confluence's v2 space operations.
  - `read:confluence-groups`: read which groups the person belongs to, to check force-release groups.
  - `read:page:confluence`: read a page's owner for a concurrent-upload notice, find the space's "Checkout notifications" page, and read the app's content property on a page.
  - `write:page:confluence`: create the "Checkout notifications" page, and write or delete the app's content property on a page.
  - `read:attachment:confluence`: list a page's attachments and read an attachment's current version.
  - `read:content.restriction:confluence`, `read:confluence-content.all`: read a page's edit restriction, in Enforced mode.
  - `write:content.restriction:confluence`, `write:confluence-content`: add the holder, the admin group and the app to a page's edit restriction, and remove them again, in Enforced mode.
  - `write:comment:confluence`: post notices as comments on the "Checkout notifications" page.
  - **Declared but not used by this version:** `report:personal-data` (the app never calls Atlassian's personal data reporting API; see the [privacy policy](./PRIVACY_POLICY.md#account-closure)), `write:attachment:confluence` (it never changes attachments), `read:comment:confluence` (it never reads comments), `read:confluence-props` and `write:confluence-props` (it reads and writes its content property through Confluence's v2 page API), `read:confluence-space.summary` (it reads spaces through Confluence's v2 API), and `read:confluence-content.summary`.
- **What the hash chain is.** Each audit record carries a SHA-256 hash of its contents and of the record before it, and Forge storage keeps a marker of the newest record. A check over the chain would show whether records were changed, removed or reordered. This version has no screen or function that runs that check or shows the audit records. It is a hash chain, with no digital signature.
- **What the pseudonyms are.** The audit log names people by an HMAC-SHA256 pseudonym of their account ID, made with a random key the app creates once per installation and keeps as a Forge storage secret. The account ID is kept beside it in the actor directory, so the pseudonym does not hide who acted.

## Encryption

The app stores its data in Forge SQL, Forge storage and your Confluence site, and sends nothing outside Atlassian's platform. Encryption in transit and at rest is provided by Atlassian as part of your site, as described in the [Atlassian Trust Center](https://www.atlassian.com/trust). The app adds no encryption of its own. Its one secret, the pseudonym key, is kept as a Forge storage secret.

## Vulnerability management

- **Reporting.** Email security@evidencepair.com. Tell us the app, what you found and how to reproduce it. Please do not test against sites you do not own. We acknowledge every report within one business day.
- **Response.** EvidencePair adopted a written security incident response plan on 12 October 2026. It covers triage, containment, investigation, notification of Atlassian and affected customers, and recovery.
- **Dependencies.** The app's npm dependencies are pinned in its lockfile. Atlassian maintains and patches the Forge runtime.
- **Bug bounty.** We do not run a bug bounty programme.

## Certifications

EvidencePair holds no security certifications, and the app has had no independent penetration test or external audit. The infrastructure the app runs on is Atlassian's, and Atlassian's own certifications cover it: see the [Atlassian Trust Center](https://www.atlassian.com/trust).
