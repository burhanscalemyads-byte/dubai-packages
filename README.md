# Glabol Dubai: landing page

A static landing page for Google Ads lead capture on Dubai tour packages. There's no build step: open `index.html` or upload the folder anywhere.

```
index.html     the whole page (HTML, CSS and JS in one file)
assets/        Glabol logo
_headers       Cloudflare Pages headers
_redirects     Cloudflare Pages redirects (keeps README private)
.htaccess      the same rules for Hostinger (Apache / LiteSpeed)
```

Photos load from Unsplash and the hero video from Pexels (both free licences). For speed and reliability, host your own copies of the two videos and change the `HERO_VIDEO` URLs.

## Before going live

All settings are in `window.GLABOL_DUBAI` at the top of `index.html`.

1. **PACKAGES**: the six Dubai trips, prices, activities and day-by-day plans are **placeholders**. Replace them with Glabol's real packages. The cards, filters, popup and "Starting at ₹..." price all update from this list.
   Each package has: `nights`, `hotel` (3/4/5), `transfers` (Shared/Private), `price`, `tags` (honeymoon/couple/family/friends), `flag` (ribbon text, or `""`), `img` (Unsplash photo id), `activities`, and `plan` (one `[title, text]` per day).
2. **LEAD_ENDPOINT**: the CRM web-form or webhook URL. Each lead is POSTed as JSON. While it is empty the page runs in preview mode (yellow note, nothing sent).
3. **GTM_ID**: Glabol's Google Tag Manager container.
4. **RATING / REVIEWS / SINCE**: check they match Glabol's Google listing today.
5. **PHONE / WHATSAPP**: digits with country code. Confirm the WhatsApp number.
6. **EMI_MONTHS**: leave at `0` unless no-cost EMI is really offered. When set (e.g. `6`), each card shows "₹X/m with no cost EMI".
7. Inclusions, exclusions, FAQ and "Good to know" copy is written in the HTML. Check it with Glabol ops (visa included? hotel class in the base price?).

## Ad URL options

The headline and filter match the ad group:

| URL | Headline | Filter |
|---|---|---|
| `?trip=honeymoon` | Dubai Honeymoon Packages | Honeymoon |
| `?trip=family` | Dubai Family Packages | Family |
| `?trip=luxury` | Dubai Luxury Packages | 5 star |
| `?trip=budget` | Dubai Packages Under ₹50k | Below ₹50k |
| (none) | Dubai Tour Packages | none |

## Leads

- The form in the hero (on phones, right under the video).
- "Customize" on any package, "Request callback", "Plan my custom trip" and "Get free quote" all open the same form in a popup. Customize attaches the package to the lead.

Two posts can share one `lead_id`:
- `stage: "lead"`: name, phone (+91XXXXXXXXXX), travel_month, travellers, package_id / package_name, form_location
- `stage: "details"` (optional, after the thank-you): hotel_category, flying_from

Both include tracking: gclid, gbraid, wbraid, utm_source, utm_medium, utm_campaign, utm_term, utm_content, trip_variant, page_url, referrer. Upsert by `lead_id`.

## dataLayer events (for GTM)

| Event | When |
|---|---|
| `generate_lead` | once, after a lead is saved. Carries `lead {id, package_id, travel_month, travellers, value, currency}` and `user_data {phone_number}` for enhanced conversions. Use it as the Google Ads conversion trigger, with transaction ID = `lead.id`. |
| `lead_form_start` | first tap into a form |
| `lead_details` | optional hotel / city answers saved |
| `cta_click` | Customize / Request callback / other quote buttons |
| `itinerary_open` | day-by-day plan opened |
| `filter_change` | a package filter switched on or off |
| `call_click`, `whatsapp_click`, `lead_post_error` | as named |

There is no thank-you page redirect, so don't also fire a page-view conversion.

## Hosting

GitHub Pages from the `main` branch (root), like the Kashmir and Himachal pages, at **https://dubai.glabol.com/**. The `CNAME` file in this repo holds the domain.

Cloudflare DNS record for glabol.com:

| Type | Name | Target | Proxy status |
|---|---|---|---|
| CNAME | dubai | burhanscalemyads-byte.github.io | Proxied (orange cloud) |

## Test locally

```
python3 -m http.server 8790
```
Then open http://localhost:8790/?gclid=TEST. In preview mode each payload is printed in the browser console.
