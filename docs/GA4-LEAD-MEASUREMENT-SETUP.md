# iHost main website — GA4 lead-measurement setup (DRAFT)

**Scope:** `https://ihost604.ca/` and its own service/vendor pages only. Do not modify the agency-operated `services.ihost604.ca`.

**Status:** Draft website-side event instrumentation only; **nothing is live** until this PR is approved and the GTM/GA4 accounts are configured and tested.

## Verified inventory (2026-11-22)
- The main website has a Google Tag Manager snippet for **GTM-5Z8R9HTD**, on its customer-facing pages.
- **Owner-verified GA4 measurement ID:** `G-FMSTQRQQ8J` for the `iHost web - GA4` property. Its Google Analytics tag is present under GTM `GTM-5Z8R9HTD` with an `Initialization - All Pages` trigger. GA4 Realtime reported visitor activity after a website visit; attribution of one specific user is not independently confirmed.
- An earlier Google setup email refers to `GTM-5KV8TKL7`, but the owner’s GTM screenshot confirms `GTM-5Z8R9HTD` belongs to iHost and is installed on the site. **Keep the existing container; do not install or duplicate the older emailed ID.**
- The owner supplied a new **Tally 4aJgbX** Standard embed snippet with **`formEventsForwarding=1`** after enabling **Track form events**. This draft adds that exact parameter to the embedded form URL on all nine pages, preserving the pre-existing embed height `2100`, title, background, align, dynamic sizing and script. It changes only HTML source attributes, not the visual design.
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

1. **Confirmed:** Use the existing iHost-owned Google Tag Manager container `GTM-5Z8R9HTD`. The earlier `GTM-5KV8TKL7` email is not a reason to install another container.
2. **Confirmed:** Use the current `iHost web - GA4` Google Analytics property and the owner-verified measurement ID `G-FMSTQRQQ8J`. Its Realtime overview has reported active users. Confirm the exact data-stream URL in GA4 Admin, if necessary.
3. **Confirmed:** The existing Google Analytics tag for `G-FMSTQRQQ8J` fires on `Initialization - All Pages` in GTM. Do not create a duplicate GA4 base tag or add a separate `gtag.js` snippet.
4. **Configure GA4 event tags in GTM.** Each event tag uses the verified stream's Google tag and the Custom Event trigger matching the table above. Map `page_path` and other fixed, non-PII parameters as needed. Only `generate_lead` should be considered the completed-form primary key event initially.
5. **Confirmed embed option:** The owner enabled **Track form events** for the Tally Standard embed and provided the generated code with `formEventsForwarding=1`. This draft adds the parameter on all nine pages. After publication, test whether `Tally.FormSubmitted` reaches the parent website. For standalone `tally.so/r/4aJgbX` links, same-session tracking needs separate handling and is not included in this release.
6. **Test before publishing GTM.** Use Tag Assistant Preview on `ihost604.ca` and GA4 DebugView; verify page views, a phone/WhatsApp click, and **one genuine test Tally submission**. Verify a single `generate_lead` event is received, not two, and that it uses the right form. The test submission must be clearly marked as a test in internal lead reports.
7. **Publish the GTM container only after reviewing all test outcomes and privacy/consent requirements.** In GA4 Admin, mark `generate_lead` as a **key event** (Google Ads conversions can be considered later when independent ads are ready). Keep micro-interaction events separate from lead conversion counts.
8. **Google Ads remains separate.** Do not link or alter the agency's Google Ads account without specific owner approval.

## SMART milestone
- **Specific:** Track site visits, quote intent, call/WhatsApp intent, and successful Tally submissions on `ihost604.ca`.
- **Measurable:** One verified GA4 stream, one confirmed `generate_lead` per completed test form, call click in DebugView, event names reported by landing page.
- **Achievable:** Google container, Analytics property and tracking-enabled Tally snippet are now verified. Remaining blockers are PR approval, GTM event-tag configuration, Tally end-to-end testing, and any necessary consent checks.
- **Relevant:** Supports a data-driven 6–12 month move from agency dependence.
- **Time-bound:** Complete account checks and a successful end-to-end measurement test within the next 7 days *after the necessary account access is available*.

**Important:** `generate_lead` means completed quote request. Qualification, booking, completed jobs, revenue and cost per booked job require a separate CRM or manual reconciliation of enquiries with actual job records.

References:
- https://tally.so/help/track-form-events-with-google-tag-manager-or-meta-pixel
- https://support.google.com/tagmanager/answer/13034206
- https://support.google.com/analytics/answer/9267735

## Tally embed comparison (confirmed against the user-provided exact snippet)

- Original URL: `https://tally.so/embed/4aJgbX?alignLeft=1&hideTitle=1&transparentBackground=1&dynamicHeight=1`
- Updated URL: `https://tally.so/embed/4aJgbX?alignLeft=1&hideTitle=1&transparentBackground=1&dynamicHeight=1&formEventsForwarding=1`
- The user’s freshly generated embed shows an initial `height="200"`; the existing website uses `height="2100"` with dynamic height. Keep **2100** to avoid the appearance/initial-height change. The Tally `widgets/embed.js` script already exists on each page, so no replacement is needed.
- Change the `/assets/site.js` query-string version on the nine pages to `?v=20261122leadtracking` so returning visitors receive the updated event listener instead of an older cached copy.
- The privacy page previously said Analytics reporting was not configured, which no longer matched the confirmed GA4 property. This draft corrects that text without changing its layout.
- **Avoid double counting:** In GTM, configure a custom-event trigger for **`generate_lead`** after the website listener pushes it. Do not separately map **`Tally.FormSubmitted`** to GA4's **`generate_lead`** event. One submission should appear exactly once in DebugView.

**Publication remains unapproved and tracking not live until GitHub PR #6 is explicitly approved and any needed Google Tag Manager changes are configured/published separately.**
