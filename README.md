# Glabol Dubai: landing page

A static landing page for Google Ads lead capture on Dubai tour packages. There's no build step: open `index.html` or upload the folder anywhere. Leads and tracking work the same way as the Kashmir and Himachal pages.

```
index.html                    the whole page (HTML, CSS and JS in one file)
assets/                       Glabol logo
tools/google-sheet-leads.gs   Apps Script for a Dubai lead sheet (see Leads)
_headers, _redirects          Cloudflare Pages rules (keep README and tools private)
.htaccess                     the same rules for Hostinger (Apache / LiteSpeed)
CNAME                         dubai.glabol.com for GitHub Pages
```

Photos load from Unsplash and the hero video from Pexels (both free licences). For speed and reliability, host your own copies of the two videos and change the `HERO_VIDEO` URLs.

## Before running ads

All settings are in `window.GLABOL_DUBAI` at the top of `index.html`.

1. **PACKAGES**: packages start at **₹45,000** per person (the 3-night Weekend Getaway). The other five trips, their prices and the day-by-day plans are still **placeholders**: replace them with Glabol's real packages. The cards, popup, "Starting at ₹..." price (always the cheapest package) and the budget question all update from this list.
   Each package has: `nights`, `hotel` (3/4/5), `transfers` (Shared/Private), `price`, `tags` (honeymoon/couple/family/friends), `flag` (ribbon text, or `""`), `img` (Unsplash photo id), `activities`, and `plan` (one `[title, text]` per day).
2. **SHEET_ENDPOINT**: leads currently go to the **Kashmir sheet**, the way Himachal ran before it had its own (see Leads).
3. **RATING / REVIEWS / SINCE**: check they match Glabol's Google listing today.
4. **WHATSAPP**: `919203405171`, opening with "Hi Glabol! I'm planning a trip to Dubai. Please share your Dubai package details and prices." (`WHATSAPP_TEXT`). The page has no phone number or call buttons, by choice: visitors use the form or WhatsApp.
5. **EMI_MONTHS**: leave at `0` unless no-cost EMI is really offered. When set (e.g. `6`), each card shows "₹X/m with no cost EMI".
6. Inclusions, exclusions, FAQ and "Good to know" copy is written in the HTML. Check it with Glabol ops (visa included? hotel class in the base price?).

## Ad URLs

Google Ads final URL suffix, the same as the other destination pages:

```
utm_source=google&utm_medium=cpc&campaign_id={campaignid}&adset_id={adgroupid}&ad_id={creative}&kw={keyword}
```

Add `trip=` to the final URL to match the headline to the ad group. All six packages always show; the matching ones move to the top:

| URL | Headline | Shown first |
|---|---|---|
| `?trip=honeymoon` | Dubai Honeymoon Packages | Honeymoon packages |
| `?trip=family` | Dubai Family Packages | Family packages |
| `?trip=luxury` | Dubai Luxury Packages | 5-star packages |
| (none) | Dubai Tour Packages | the usual order |

## Leads

Every enquiry is sent to the **Google Sheet** and the **Glabol CRM** at the same time, like the other destination pages. The forms are the hero form (on phones, right under the video) and the popup that "Customize", "Request callback", "Plan my custom trip" and "Get free quote" open. Customize attaches the package to the lead.

**Google Sheet** (`SHEET_ENDPOINT`): one row per enquiry with the same columns as the Kashmir and Himachal sheets:

| Timestamp | Name | Phone | Travel month | Travellers | Lead quality | Budget fit | Booking timeline | Best time to call | Source | Medium | Campaign ID | Adset ID | Ad ID | Keyword | GCLID | Landing page | Form | Lead ID |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

- **Form** says where the enquiry came from and the package, e.g. `Dubai hero` or `Dubai popup - Dubai Honeymoon 4N/5D`.
- **For now the URL is the Kashmir sheet.** Filter its "Form" or "Landing page" column for Dubai. Its email alerts still say "New Kashmir lead".
- **To give Dubai its own sheet** (about 5 minutes): follow the steps at the top of [`tools/google-sheet-leads.gs`](tools/google-sheet-leads.gs). That copy already says "Dubai" in its alerts. Put the new web app URL in `SHEET_ENDPOINT`.

**Glabol CRM** (`CRM_WEBHOOK_URL`, `CRM_API_KEY`, `DESTINATION`): JSON with an `x-api-key` header, the same as the other pages:

| Field | Value |
|---|---|
| `name`, `phone` | from the form; phone as +91XXXXXXXXXX |
| `email` | empty |
| `destination` | `Dubai` |
| `message` | "Interested in Dubai package: Classic Dubai 4N/5D. Travel month: Dec 2026. Travellers: 2 people." |
| `host` | the page's domain, e.g. dubai.glabol.com |
| `source` | "Google Ads" for Google ad clicks, "Meta Ads" for Facebook or Instagram, "Website" otherwise |

The CRM accepts requests from `*.glabol.com` pages only. If it's down or slow, the visitor still gets the thank-you (the page waits at most 5 seconds) and the lead is still in the sheet.

**Thank-you questions:** after an enquiry the card asks the same three questions as the other pages (budget fit, booking timeline, best time to call). Each answer updates the lead's sheet row and sets Lead quality: Hot (budget fits, booking within a month), Warm (budget fits, just exploring), Cold (needs something cheaper) or Partial.

## Ad tracking (GTM-TTXFHSF9)

Every ad tag lives in Glabol's GTM container, shared with the other destination pages; the page loads no pixel itself. It pushes the same events with the same data, so the existing tags work unchanged:

| Event | When | Data |
|---|---|---|
| `lead_form_submit` | once per enquiry, after it is sent | `lead_id`; `user_data.phone_number` (+91…) |
| `thank_you_page_view` | straight after it | `lead_id`, `transaction_id` (= lead ID) |
| `qualified_lead` | once, when the answers make the lead Hot | `lead_id`, `budget_fit`, `booking_timeline` |
| `lead_questions_complete` | once, when all three questions are answered | `lead_id`, `lead_quality`, `budget_fit`, `booking_timeline`, `call_time` |

The page also pushes `lead_form_start`, `cta_click`, `itinerary_open`, `whatsapp_click` and `lead_post_error`. No GTM tags use them yet.

WhatsApp chats started from the page aren't recorded in the sheet or CRM, unlike form enquiries. `whatsapp_click` is there if you want GTM to count them.

## Test

Local copies never send leads and don't load GTM, so tests can't fill the sheet or the CRM, or count as conversions. Each payload is printed in the browser console instead.

```
python3 -m http.server 8791
```

Open http://localhost:8791/?utm_source=google&utm_medium=cpc&campaign_id=111&adset_id=222&ad_id=333&kw=dubai+tour+package&gclid=test123

On the live site, add `?nosend=1` to test the same way. To check the full path once, send one real enquiry on https://dubai.glabol.com/ named "Test lead (delete me)", then delete it from the sheet and the CRM.

## Hosting

GitHub Pages from the `main` branch (root), like the Kashmir and Himachal pages, at **https://dubai.glabol.com/**. The `CNAME` file holds the domain.

Cloudflare DNS record for glabol.com:

| Type | Name | Target | Proxy status |
|---|---|---|---|
| CNAME | dubai | burhanscalemyads-byte.github.io | Proxied (orange cloud) |
