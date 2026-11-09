# Privacy policy: Attachment Checkout for Confluence

Updated 9 November 2026

This policy describes the Atlassian Marketplace app **Attachment Checkout for Confluence** ("the app"), made by EvidencePair LLC ("EvidencePair", "we"). EvidencePair's company privacy policy, which covers our website and email you send us, is at <https://evidencepair.com/privacy/>.

## In short

- The app runs on Atlassian Forge, inside your Confluence site. It declares no external egress and no remote services. Apart from its own Forge storage, it makes requests only to your site's Confluence APIs.
- It keeps its records in Forge SQL and Forge storage, which Atlassian hosts for the app. It also writes to your Confluence site: a lock count on pages with checked-out attachments, page edit restrictions in Enforced mode, and notices on a "Checkout notifications" page. None of this is sent to EvidencePair.
- The personal data it stores is Atlassian account IDs, plus the notes, check-in comments and force-release reasons people type.
- It does not report personal data to Atlassian, and it erases nothing when an Atlassian account is closed.

## What the app stores

### In Forge SQL

**Checkouts (`aco_locks`), one row per checkout, kept after the checkout ends:**

- the page ID, space key and attachment ID;
- the Atlassian account ID of the person who checked the attachment out (the holder), and their note, if they wrote one;
- when it was checked out and when it expires;
- the attachment's version number and version time at check-out, and again at check-in;
- how the checkout ended (checked in, force-released, expired, or closed because the attachment or page was trashed or deleted), when, and who ended it: the account ID of the person who checked it in or force-released it, or a label for the app's hourly job or a Confluence event;
- the space's mode at check-out (Advisory or Enforced).

**Checkout events (`aco_events`), one row per event:**

- a new version of a checked-out attachment uploaded by someone other than the holder: the uploader's account ID, the version number and the version time;
- a checkout closed because its attachment was trashed or deleted, with the account ID that Confluence's event names, or because its page was trashed or deleted;
- a reminder sent about a day before a checkout expires, with the expiry time.

Each row also holds the checkout's ID, the event type and when the app recorded it.

**Space settings (`aco_space_settings`), one row per space where an admin saved settings or the app posted a notice:**

- the space key and ID, whether checkout is on, the mode, and the default and maximum checkout durations;
- the admin group ID and the force-release group IDs;
- the ID of the space's "Checkout notifications" page;
- when the settings were last saved, and the account ID of the admin who saved them.

**Enforced-mode restriction state (`aco_page_restrictions`), one row per page with an Enforced checkout:**

- the page's edit restriction as it was before the first Enforced checkout: the account IDs and group IDs in it. These can be account IDs of people who never use the app;
- the entries the app added: holders' account IDs, the app's own account ID and the admin group ID;
- whether the app's own entry was left on the page.

**Pages to refresh (`aco_stale_summaries`):** the IDs of pages whose lock count the app could not update.

**Audit log (`audit_events`), one record per check-out, check-in, force release, expiry, concurrent upload and closure:**

- when it happened, the action, the attachment ID, a label naming the attachment ID and page ID, and the outcome;
- who did it, as a pseudonym: an HMAC-SHA256 of the person's account ID, made with a random key the app creates once per installation. For the hourly job and Confluence events, a pseudonym of a system label;
- details that depend on the action. A check-out records the mode, expiry, version and note. A check-in records the versions and the check-in comment. A force release records the holder's account ID and the reason (the first 4,000 characters). An expiry records the holder's account ID. A concurrent upload records the holder's and the uploader's account IDs and the version. A closure records the event type and the account ID that Confluence's event names. Check-ins, force releases and expiries also record whether restriction entries were left to remove;
- a SHA-256 hash that chains each record to the one before it.

**Actor directory (`audit_actors`), one row per person who has checked out, checked in or force-released:** the pseudonym and the account ID it stands for. The app records no display names.

The database also holds two tables, `jobs` and `job_items`, that come with the app's shared code. This app writes nothing to them.

### In Forge storage

- the random key for the audit pseudonyms, kept as a Forge storage secret;
- a marker of the newest audit record (its number and hash), and a marker for an audit record while it is being written;
- a hold on a page while one change to its checkouts runs: a random ID and an expiry time. The hold lasts 60 seconds at a time and is deleted when the change finishes.

None of these hold account IDs or text people type.

### In your Confluence site

- **A content property, `aco-lock-summary`,** on each page with checked-out attachments. It holds the byline text ("🔒 2 checked out") and a tooltip that says how many attachments are checked out.
- **Edit restriction entries, in Enforced mode.** At check-out the app adds the holder, the space's admin group and the app's own account to the page's edit restriction, each only if it is not already there.
- **A page titled "Checkout notifications"** in each space where the app posts a notice or an admin saves settings with checkout turned on.
- **Comments on that page**, posted as the app. A concurrent-upload notice @mentions the holder, the page's owner and the uploader, and gives the attachment ID, page ID and version number. A force-release notice @mentions the holder and the person who released it, and gives the reason. An expiry notice and the day-before reminder @mention the holder and give the expiry time.

The app does not look up names, email addresses or profiles. It stores attachment and page IDs, not their titles. Notes, check-in comments and force-release reasons can contain personal data, and the app stores them exactly as entered.

## What the app reads from Confluence

- **As the person using it:** whether they can view or edit the page; whether they are a Confluence admin or an admin of the space; and, when a space has force-release groups, which groups they belong to.
- **As the app:** the page's attachments (ID, title, size, type, download link, and current version number, time and author ID), to show in its window; an attachment's current version; the page's edit restriction; the app's own account ID; a page's owner, for a concurrent-upload notice; the space's ID; whether the "Checkout notifications" page exists; and its own content property.
- **Confluence events:** when an attachment is created, updated, trashed or deleted (the attachment ID, page ID, version, and the account ID of the person who did it), and when a page is trashed or deleted (the page ID).

It reads no attachment contents, page bodies or comments.

## Where the data lives

Atlassian hosts Forge SQL and Forge storage for the app on its own infrastructure, as part of the Forge platform, under Atlassian's own security and privacy commitments. The content property, restriction entries, notifications page and comments are part of your Confluence site's data. Atlassian stores them with the rest of your site, in your site's data residency location. EvidencePair runs no servers in the app's path.

## Who can see it

- **Anyone who can view a page** can open the app's window there. The window shows the page's active checkouts (each holder's account ID, note, times and versions) and any concurrent uploads with the uploader's account ID. It also shows an attachment's full checkout history: every past checkout with its holder, note, who ended it and when, and its events. The window also receives the space's settings, including the account ID of the admin who last saved them.
- **A gap in that check.** The app checks that the person can view the page named in the request. It does not check that the attachment, or the space whose settings it returns, belongs to that page. Someone who can view any page can therefore read the checkout history of any attachment, and the settings of any space, if they know its ID or key.
- **Confluence admins** can see every active checkout on the site, with holders and notes, in the app's overview. They can export it as a CSV with the space key, page ID, attachment ID, holder's account ID, mode, check-out and expiry times, version and note.
- **Everyone signed in** sees their own active checkouts under "My checkouts", and can export those.
- **Space admins and Confluence admins** see the space's settings screen.
- **No screen in the app** shows the audit log or the actor directory.
- **Confluence's permissions** decide who sees the "Checkout notifications" page and its comments, a page's edit restriction, and the content property. The content property is readable by anyone who can view the page, through Confluence's REST API, and holds only the count.

## Retention and deletion

- **Checkouts, checkout events, space settings, the audit log and the actor directory** are kept with no time limit. The app has no retention setting and no function to delete them, and it deletes none of them on a schedule. An ended checkout stays as history. Trashing or deleting a page or attachment closes its checkouts and keeps their records. The only checkout rows the app deletes come from a check-out that failed part-way.
- **Short-lived records** go on their own. A page's restriction state is deleted once the app has removed everything it added, or when the page is trashed or deleted. A page leaves the refresh list once its lock count is rewritten. Page holds and audit write markers are deleted as soon as their step ends.
- **The content property** is deleted when the last checkout on the page ends.
- **Restriction entries** the app added are removed when the page's Enforced checkouts end. If a removal does not go through, the entry stays, and the app tries again when a later Enforced checkout on that page ends.
- **The "Checkout notifications" page and its comments** are never deleted by the app. People with permission can delete them like any other page or comment.
- **Uninstalling.** The app has no uninstall step, so it deletes nothing when it is uninstalled. What then happens to its Forge SQL and Forge storage data is decided by Atlassian's platform. What it wrote to Confluence, such as the notifications page and its comments, stays in your site until someone deletes it.

## Account closure

The app declares Atlassian's `report:personal-data` scope, but this version never calls Atlassian's personal data reporting API. It does not tell Atlassian which account IDs it stores, it never learns that an account has been closed, and it erases nothing when one is. A closed account's ID stays in every record listed above until that record is deleted, and in the comments the app posted until someone deletes them.

The audit log's actor column holds a pseudonym in place of the account ID. That does not hide the person: the actor directory maps each pseudonym to its account ID, and some audit records name account IDs in their details. The app's code includes a routine that would blank a closed account in the actor directory, but nothing in this version runs it.

## Platform logs

The app writes short lines to the Forge platform's logs:

- when a request is refused or fails: the action's name, an error code and the error's details. The details can include the Confluence request that failed (its path, which can contain a page ID, attachment ID, space key or account ID), its HTTP status, and up to 2,000 characters of Confluence's answer to a request it rejected as invalid;
- when a background step fails, such as posting a notice or updating a lock count: the step, the checkout or page ID, and the error;
- when the app sets up its database tables: how many changes it applied.

In the app's development environment only, it also logs one line per call with the function name, a random request ID, the build and a short installation tag. The app does not write notes, check-in comments or force-release reasons to its logs. Atlassian's platform can also record errors it sees while the app runs, such as an unhandled error and its message. Atlassian decides who can read these logs, and the app's developer can read them under Atlassian's rules ([Atlassian: view app logs](https://developer.atlassian.com/platform/forge/view-app-logs/)).

## Email and support

If you email us, our [company privacy policy](https://evidencepair.com/privacy/) describes how we handle that email, and the current restrictions on where our apps may be used. For help with the app, see <https://evidencepair.com/support/>.

## What we never do

We do not sell personal data or use it for unrelated advertising or AI training. The app sends none of its data to us.

## Changes to this policy

The date at the top changes when this policy does. Every version is kept in this repository's history.

## Contact

hello@evidencepair.com
