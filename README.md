# Attachment Checkout for Confluence

**100% Native Forge · Runs on Atlassian (Zero Remotes / Zero Data Egress) · Enterprise Document Control for Confluence Cloud**

Attachment Checkout for Confluence brings structured check-out / check-in locking, configurable auto-expiry, `@mention` collaboration notices, space-level governance, and 1-click ISO 27001 / SOC 2 CSV audit exports to Confluence Cloud attachments.

Built from the ground up on **Atlassian Forge** (`nodejs24.x` runtime, Forge SQL, and Atlassian KVS) with **zero external remotes and zero data egress**, Attachment Checkout is designed for teams migrating off legacy Atlassian Connect attachment-locking apps ahead of Atlassian Connect End of Support (**January 31, 2027**) as well as regulated organizations that require stronger governance than basic Forge file locks.

---

## Four Enterprise Capabilities

1. **Automated Lock Expiry**
   Configurable default (e.g., 3 days) and maximum (up to 365 days) checkout durations per space ensure that vacationing or departing teammates never permanently block page attachments. An hourly scheduled job automatically expires stale locks, sends a 24-hour pre-expiry reminder, and reconciles page lock summaries.
2. **Real-Time `@Mention` Notifications**
   Instead of cluttering content pages or sending external emails, Attachment Checkout maintains a dedicated **"Checkout notifications"** page per space and posts storage-format `@mention` comments for concurrent-upload violations, 24-hour pre-expiry warnings, auto-expirations, and admin force-releases.
3. **Space-Level Governance & Force-Release (`Advisory` & `Enforced` Modes)**
   - **Advisory Mode (Default):** Collaborators see who holds a checkout, when it expires, and their checkout note; concurrent uploads by other users are detected via Confluence product events, recorded in the attachment's audit history, and flagged with a warning banner.
   - **Enforced Mode:** Applies a Confluence page edit restriction while any attachment on the page is checked out, preventing conflicting attachment uploads and restoring the page's exact prior restrictions on release.
   - **Granular Force-Release:** Confluence admins, space admins, and delegated force-release groups can unlock any file with a mandatory audit reason.
4. **Compliance CSV Audit Export**
   1-click UTF-8 CSV export (with formula-injection protection) of active checkouts and lock metadata for ISO 27001, SOC 2, and GxP document-control audits.

---

## Documentation Links

- **[User & Administrator Guide](./USER_GUIDE.md)** — Installing, configuring space governance, checking files out/in, force-releasing locks, and exporting CSV audits.
- **[Privacy, Security & Runs on Atlassian Architecture](./PRIVACY_AND_SECURITY.md)** — Zero-egress declaration, Forge SQL data residency, permission scopes, and personal-data lifecycle.
- **[End-User License Agreement (EULA)](./EULA.md)** — Standard commercial terms for EvidencePair LLC Marketplace apps.
- **[Marketplace Listing Copy & Highlights](./LISTING_COPY.md)** — Full Marketplace overview, highlights, and positioning.
- **[Pricing Table](./PRICING_TABLE.md)** — Progressive per-user monthly Cloud pricing tiers.

---

## Support & Vendor Contact

- **Vendor:** EvidencePair LLC
- **Support:** `support@titlecovenant.com`
- **Documentation Repository:** <https://github.com/Allerdyce/confluence-attachment-checkout-docs>
