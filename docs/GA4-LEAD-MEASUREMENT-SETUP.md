# iHost main website — GA4 lead-measurement setup (DRAFT)

**Scope:** `https://ihost604.ca/` and its own service/vendor pages only. Do not modify the agency-operated `services.ihost604.ca`.

**Status:** Draft website-side event instrumentation only; **nothing is live** until this PR is approved and the GTM/GA4 accounts are configured and tested.

## Verified inventory (2026-11-22)
- The main website has a Google Tag Manager snippet for **GTM-5Z8R9HTD**, on its customer-facing pages.
- The repository has **no verified GA4 web stream Measurement ID (`G-...`)**, and a GTM container snippet by itself does not establish that GA4 collects data.
- A Google setup email sent on 2026-11-15 to `ihost604@gmail.com` refers to a **different** ID, **GTM-5KV8TKL7**. Do **not** add it to the site or overwrite the existing container until ownership, destination, and purpose have been verified.
- Customer quote forms are Tally **4aJgbX** (embedded across nine pages). Existing code is an iframe embed; no confirmed 'Track form events' option was detected in the embed snippet.
- The main site already pushes `click_call` and `quote_open` to `window.dataLayer`.
- Tally also offers a standalone new-tab form link. Events fired on the standalone `tally.so` page will **not** automatically be attributed to this website's GA4 session.
- No payment, advertising budget, or new website appearance is involved in this change.

## Draft event contract (website JavaScript)

| dataLayer event | Trigger | Meaning | GA4 event name |
| --- | --- | --- | --- |
| `quote_open` | Click on the quote CTA | Interest only; not a lead | `quote_open` (custom) |
| `quote_form_external_open` | Click on standalone Tally new-tab link | Interest only; not a lead | `quote_form_external_open` (custom) |
| `click_call` | Tap/click a `tel:` link | Call intent, **not** an answered call | `click_call` (custom) |
| `click_whatsapp` | Tap/click WhatsApp link | Contact intent, **not** a conversation | `click_whatsapp` (custom) |
| `generate_lead` | Tally emits `Tally.FormSubmitted` from the embedded form | **Successful embedded quote submission**; not a qualified or booked job | `generate_lead` (recommended) |

The `generate_lead` listener only passes the fixed parameters `lead_type=quote_request`, `form_provider=tally`, `form_id=4aJgbX`, plus the existing `page_path`. No names, addresses, email addresses, phone numbers, form answers, pictures, or free-text values are sent by this event.

Deduplication: the website ignores duplicate `Tally.FormSubmitted` notifications within 2 seconds. Do not also configure another GTM trigger to send the same `generate_lead` event directly from `Tally.FormSubmitted`; that would double-count.

## Configuration to complete in the owner's Google account

1. **Identify the correct Tag Manager container.** Log in at `https://tagmanager.google.com/` with the iHost-controlled account; confirm `GTM-5Z8R9HTD` is under iHost ownership. Investigate why Google emailed a different ID `GTM-5KV8TKL7`. Do not install two containers accidentally.
2. **Create or identify an iHost-owned GA4 property.** Use `https://analytics.google.com/` -> Admin -> Data streams. Web stream URL must be `https://ihost604.ca/`. Record the real Measurement ID `G-...`; no placeholders or guesses in production code.
3. **Configure Google tag in the verified GTM container.** Tag type: Google tag; ID = real `G-...`; trigger = Initialization/All Pages as suitable for the consent setup. Do not paste an additional hardcoded `gtag.js` snippet into the site.
4. **Configure GA4 event tags in GTM.** Each event tag uses the verified stream's Google tag and the Custom Event trigger matching the table above. Map `page_path` and other fixed, non-PII parameters as needed. Only `generate_lead` should be considered the completed-form primary key event initially.
5. **Enable 'Track form events' for the Tally embed.** In the Tally form's Share -> Embed settings choose **Track form events** and obtain the exact tracking-enabled snippet. Preserve the existing styling. Its events should reach the parent site. If that option generates different embed markup, prepare a separate, owner-approved code change; do not invent Tally attributes. For standalone Tally forms, consider a verified custom domain or separate Tally GA4 setup later.
6. **Test before publishing GTM.** Use Tag Assistant Preview on `ihost604.ca` and GA4 DebugView; verify page views, a phone/WhatsApp click, and **one genuine test Tally submission**. Verify a single `generate_lead` event is received, not two, and that it uses the right form. The test submission must be clearly marked as a test in internal lead reports.
7. **Publish the GTM container only after reviewing all test outcomes and privacy/consent requirements.** In GA4 Admin, mark `generate_lead` as a **key event** (Google Ads conversions can be considered later when independent ads are ready). Keep micro-interaction events separate from lead conversion counts.
8. **Google Ads remains separate.** Do not link or alter the agency's Google Ads account without specific owner approval.

## SMART milestone
- **Specific:** Track site visits, quote intent, call/WhatsApp intent, and successful Tally submissions on `ihost604.ca`.
- **Measurable:** One verified GA4 stream, one confirmed `generate_lead` per completed test form, call click in DebugView, event names reported by landing page.
- **Achievable:** Website code and existing GTM snippet are ready. Remaining blockers are correct Google/Tally account configuration and consent/testing.
- **Relevant:** Supports a data-driven 6–12 month move from agency dependence.
- **Time-bound:** Complete account checks and a successful end-to-end measurement test within the next 7 days *after the necessary account access is available*.

**Important:** `generate_lead` means completed quote request. Qualification, booking, completed jobs, revenue and cost per booked job require a separate CRM or manual reconciliation of enquiries with actual job records.

References:
- https://tally.so/help/track-form-events-with-google-tag-manager-or-meta-pixel
- https://support.google.com/tagmanager/answer/13034206
- https://support.google.com/analytics/answer/9267735
