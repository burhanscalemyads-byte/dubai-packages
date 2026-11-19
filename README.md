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

1. **PACKAGES**: Glabol's three Dubai packages, each with an Abu Dhabi day trip: **Dubai Essentials** 4N/5D ₹45,000, **Dubai Explorer** 5N/6D ₹55,000 and **Dubai with Atlantis** 6N/7D ₹70,000, per person. The cards, popup, "Starting at ₹..." price (always the cheapest package) and the budget question all update from this list.
   Each package has: `id`, `name`, `nights`, `route` (the line under the name), `price`, `flag` (ribbon text, or `""`), `img` (Unsplash photo id), `activities` (the first ones show on the card) and `plan` (one `[title, text]` per day). Add `hotel: 4` to show a star rating on the card. Transfers always show as just "Transfers".
2. **SHEET_ENDPOINT**: leads currently go to the **Kashmir sheet**, the way Himachal ran before it had its own (see Leads).
3. **RATING / REVIEWS / SINCE**: check they match Glabol's Google listing today.
4. **WHATSAPP**: `919203405171`, opening with "Hi Glabol! I'm planning a trip to Dubai. Please share your Dubai package details and prices." (`WHATSAPP_TEXT`). The page has no phone number or call buttons, by choice: visitors use the form or WhatsApp.
5. **EMI_MONTHS**: leave at `0` unless no-cost EMI is really offered. When set (e.g. `6`), each card shows "₹X/m with no cost EMI".
6. Inclusions, exclusions, FAQ and "Good to know" copy is written in the HTML. Check it with Glabol ops (visa included? hotel class in the base price?).

## Ad URLs

The Dubai search campaign (`SMA_Search_DubaiPackage_18.11.2026`) has this final URL suffix:

```
utm_source=google&utm_medium=cpc&utm_campaign=dubai_search&campaignid={campaignid}&adgroupid={adgroupid}&creative={creative}&kw={keyword}&matchtype={matchtype}&device={device}
```

The main headline, the form title and the first package follow the ad group (`VARIANTS` in the settings). All three packages always show. The page uses the first match:

1. `utm_content=` or `trip=` naming a variant: `family`, `honeymoon`, `cost`, `india` or `tour`
2. `adgroupid=` (or `adset_id=`) listed in a variant's `adgroups`
3. a word from a variant's `words` in `utm_content`, `trip`, `utm_campaign` or the keyword (`kw` / `utm_term`)

| Ad group (id) | Headline | Form title | Shown first |
|---|---|---|---|
| Dubai Tour Packages (203700663793) | Dubai Tour Packages | Get your Dubai plan and price | the usual order |
| Dubai Family Packages (199441917782) | Dubai Family Packages | Get your family's Dubai plan and price | 6N, ribbon "Family pick" |
| Dubai Honeymoon & Couple (203237750058) | Dubai Honeymoon & Couple Packages | Get your Dubai honeymoon plan and price | 5N, ribbon "Honeymoon pick" |
| Dubai Trip Cost (203700668553) | Dubai Trip Cost from ₹45,000 | Get your Dubai trip cost | 4N, ribbon "Lowest price" |
| Dubai Packages From India (205301354794) | Dubai Packages from India | Get your Dubai plan and price | the usual order |

No match shows "Dubai Tour Packages". For a new ad group, add its id to a variant's `adgroups`, or put `utm_content=<variant>` in its URL.

## Leads

Every enquiry is sent to the **Google Sheet** and the **Glabol CRM** at the same time, like the other destination pages. The forms are the hero form (on phones, right under the video) and the popup that "Customize", "Request callback", "Plan my custom trip" and "Get free quote" open. Customize attaches the package to the lead.

**Google Sheet** (`SHEET_ENDPOINT`): one row per enquiry with the same columns as the Kashmir and Himachal sheets:

| Timestamp | Name | Phone | Travel month | Travellers | Lead quality | Budget fit | Booking timeline | Best time to call | Source | Medium | Campaign ID | Adset ID | Ad ID | Keyword | GCLID | Landing page | Form | Lead ID |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

- **Form** says where the enquiry came from and the package, e.g. `Dubai hero` or `Dubai popup - Dubai Explorer 5N/6D`.
- **For now the URL is the Kashmir sheet.** Filter its "Form" or "Landing page" column for Dubai. Its email alerts still say "New Kashmir lead".
- **To give Dubai its own sheet** (about 5 minutes): follow the steps at the top of [`tools/google-sheet-leads.gs`](tools/google-sheet-leads.gs). That copy already says "Dubai" in its alerts. Put the new web app URL in `SHEET_ENDPOINT`.

**Glabol CRM** (`CRM_WEBHOOK_URL`, `CRM_API_KEY`, `DESTINATION`): JSON with an `x-api-key` header, the same as the other pages:

| Field | Value |
|---|---|
| `name`, `phone` | from the form; phone as +91XXXXXXXXXX |
| `email` | empty |
| `destination` | `Dubai` |
| `message` | "Interested in Dubai package: Dubai Essentials 4N/5D. Travel month: Dec 2026. Travellers: 2 people." |
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
