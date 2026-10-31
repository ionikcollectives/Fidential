# FIDENTIAL website

Static site for https://fidential.shop, hosted on Vercel. There is no build step and no backend. Orders go out on WhatsApp, and reviews go through Formspree.

## Files
The site is the `fidential-website/` folder. File paths in this README are inside it.
- `index.html`: the whole site (HTML, CSS and JavaScript).
- `img/`: product photos, hang-tag cutouts (`*_hero.webp`) and their depth maps (`*_depth.webp`), the white logo, the link-preview image (`og.jpg`), the square logo for search engines (`logo-512.png`) and app icons.
- `fonts/`: the three typefaces (Archivo, Black Ops One and Geist Mono), from Google Fonts under the SIL Open Font License.
- `vendor/`: the libraries the site runs on: Three.js for the 3D, GSAP for animation and Lenis for smooth scrolling.
- `404.html`: the "page not found" page.
- `favicon.ico`, `favicon.svg`, `apple-touch-icon.png`, `site.webmanifest`: browser and phone icons.
- `robots.txt`, `sitemap.xml`: these tell search engines what to index.
- `vercel.json`: hosting settings for Vercel (see Hosting).

Keep the files and folders in this layout. Vercel publishes the folder exactly as it is.

At the top of the repo, next to this README, `.gitignore` keeps Finder's `.DS_Store` files and Vercel's local `.vercel` folder out of GitHub, and `.vscode/settings.json` points VS Code's Live Server at the site folder.

## Reviews (Formspree)
Customers review in four steps: **Rate → Fit → Review → Send**. They pick the pair, tap a star rating, and can add their size, fit and highlights. Then they write the review and choose whether it's a **public review** or **private feedback**. A hang tag beside the form fills in live as they type, and gets stamped "Received" when the review is sent. Drafts are saved on the customer's device until they send.

### Setup
The site is connected to the Formspree form `meaalldw` (`https://formspree.io/f/meaalldw`), set as `formspree: "meaalldw",` in the `STORE` block. To use a different form later, replace that ID. The full endpoint URL works too.

In the Formspree dashboard:
1. In the form's **Settings → Spam protection**, make sure **reCAPTCHA is off**. The site sends reviews without leaving the page, and Formspree's reCAPTCHA doesn't support that. A hidden honeypot field (`_gotcha`) and Formspree's own spam filter still block bots.
2. Once the site is live, restrict the form to your domain (`fidential.shop`) in the form settings. Other websites then can't post to it. Local tests from Live Server stop working after that, which is expected.
3. Send a test review and check your inbox.

If the ID is ever removed, or Formspree can't be reached, the review opens in WhatsApp already written out. No review is lost.

The free plan allows 50 submissions a month and keeps them in the dashboard for 30 days. The emails stay in your inbox.

### What arrives
Each email has the pair, rating, headline, review, size, fit, highlights, name, email (if given) and order number (if given). The subject line starts with the stars, e.g. `★★★★★ Review: Black Panther, from Kavindu Perera`. Reply to the email to answer the customer when they gave an address.

### Publishing a review on the site
Public reviews include a line called `website_entry`:

```
{"product":"black-panther","rating":5,"title":"My go-to gym shorts","text":"…","name":"Kavindu P.","size":"M","fit":"True to size","date":"2026-10-31","verified":false},
```

1. Copy the line exactly as it is. Paste it inside `published: [ … ]` in the `STORE` block, one review per line.
2. If the order number matches a real order, change `"verified":false` to `"verified":true`. The review then shows a "Verified order" badge.
3. Publish the change (see Hosting).

When the first review is published, the reviews wall appears automatically. It shows the average score, the star breakdown, a fit summary, filters by pair, and the review cards. Star ratings also appear on the product cards and the product view, and the ratings are added to the search-engine data.

Only publish real reviews from customers who chose "Public review". You can fix typos, but keep the meaning. Leave private feedback unpublished.

**Tip:** after a delivery, send the customer `https://fidential.shop/#reviews`. The link opens the page right at the review form. If they ordered from the same phone, the pair, size and order number are filled in for them.

## Social links
Instagram (`@fidential_official`) and Facebook appear in the footer, the mobile menu and the thank-you screen after a review, and in the search-engine data at the top of `index.html`. To change a link, search `index.html` for `instagram.com/fidential_official` or `facebook.com/profile.php` and replace every match.

## The hero hang tag
The hero is a 3D clothing hang tag hanging from a cord. Visitors drag (or swipe) to spin it, tap to flip it over, and the colourway buttons spin it round to the other pair. The front shows the `*_hero.webp` cutout inside a window, with the depth map giving it real depth as the tag tilts. The back is a spec sheet.

Everything printed on the tag (name, colour, price, sizes, waist ranges) is drawn from the `STORE` block, so it stays in sync when you edit prices or sizes. If a device can't run the 3D smoothly (no WebGL, or no graphics chip, so the 3D would run in software), the hero shows the cutout on a flat card instead.

## Editing the shop
Open `index.html` and find the `STORE` block at the top of the first `<script>`:

```js
const STORE = {
  whatsapp: "94772952492",   // WhatsApp number: country code + number, no "+" or spaces
  products: {
    "black-panther": { ..., price: 2900, stock: { S: 26, M: 5, L: 1 } },
    "blue-sky":      { ..., price: 2900, stock: { S: 7,  M: 1, L: 1 } }
  },
  sizes: { ... },
  reviews: { formspree: "meaalldw", published: [ ... ] }
};
```

- **WhatsApp number**: change `whatsapp`. Also update the number in the footer (search for `+94 77 295 2492` and `wa.me/94772952492`) and the `telephone` in the search-engine data at the top of the file.
- **Prices**: change `price`. The hero, cards, cart and WhatsApp message all read from it. A few places are plain text, so search for `2,900` and `2900` and update them too: the fabric spec list, the page description at the top, and the two `"price": "2900"` lines in the search-engine data.
- **Stock**: change `stock`. Sizes at 0 show as sold out, 1 shows "Last one", and 5 or fewer show "Low stock". Customers can't add more than the stock number. If a whole pair sells out, also change its `"availability"` in the search-engine data to `"https://schema.org/OutOfStock"`.
- **Photos**: replace files in `img/` with the same names (`bp_1.webp`… for Black Panther, `bs_1.webp`… for Blue Sky). Use WebP, about 1000 px wide and under 100 KB, so pages stay fast. The hang tag uses each product's `hero` and `depth` files: a new cutout needs a matching 2:3 depth map (white = nearest), also saved as WebP.

## How checkout works
The cart runs in three steps:
1. **Bag**: change quantities or remove items, then "Check out".
2. **Details**: optional name, phone, city, address and note, plus an order summary. "Send order on WhatsApp" opens `https://wa.me/<number>?text=…` with the order number, items, sizes, quantities, total and any details already written.
3. **Sent**: confirms the order is waiting in WhatsApp, with options to open it again, copy the order text, edit the order or start a new one.

"Buy now" on a product adds it and jumps straight to step 2. The customer taps send in WhatsApp. Delivery charges and payment are confirmed in the chat.

## Search engines (SEO)
Already in place:
- A page title and description aimed at "gym shorts" and "activewear" in Sri Lanka.
- A main heading that says what the site sells, descriptive image text, and a canonical address.
- Link previews for WhatsApp, Instagram, Facebook and X, using `img/og.jpg`.
- Search-engine data (schema.org) for the brand, its Instagram, Facebook and WhatsApp, and both pairs with price and stock. Review stars are added once reviews are published.
- `sitemap.xml`, `robots.txt`, favicons and app icons.

After the site goes live:
1. **Google Search Console** (https://search.google.com/search-console): add `fidential.shop` as a **Domain** property. Google gives you a TXT record: add it in GoDaddy under **DNS → DNS Records → Add New Record** (type TXT, name `@`), then click Verify. Then submit `sitemap.xml` under **Sitemaps**.
2. **Socials**: put `fidential.shop` in the Instagram and Facebook bios.
3. **Check the preview**: paste the address into Facebook's Sharing Debugger (https://developers.facebook.com/tools/debug/) to see the preview card and refresh it.
4. **Optional, Google Merchant Center**: free product listings in Google Shopping. It asks for delivery and return details.
5. When the page changes a lot, update `<lastmod>` in `sitemap.xml`.

## Speed
- Fonts and libraries are served from the site itself (`fonts/`, `vendor/`), so the first screen needs no other servers. Browsers keep them for a year. That's safe because each file name includes its version: to update a library, add the new version under its own name and change the reference in `index.html`.
- The first screen is the intro loader. The 3D hang tag starts once the loader has painted, and the fabric scene is only built when a visitor scrolls near it.
- The full intro plays once a day. Visitors back within 24 hours get the quick version.
- The loader and the cart pill move with transforms only, so nothing on the page jumps while it loads.

Lighthouse on a local copy before launch: Performance 90–93 on mobile and 98–99 on desktop; Accessibility, Best Practices and SEO 100. Google's PageSpeed Insights varies a few points between runs.

## Hosting (GitHub → Vercel, domain at GoDaddy)
The code is in the GitHub repository `ionikcollectives/Fidential-Web`. Vercel publishes its `fidential-website/` folder at **https://fidential.shop**, and the domain is registered at GoDaddy.

- `vercel.json` adds basic security headers. It lets browsers keep images for a day, and the fonts and libraries for a year. After you replace a photo, a returning visitor can see the old one for up to a day.
- Vercel serves `404.html` for any address that doesn't exist, and sets up HTTPS by itself.
- Vercel's free Hobby plan is for non-commercial use only. Selling products counts as commercial, so the project needs the Pro plan.

### First-time setup
1. In Vercel, choose **Add New → Project** and import the `Fidential-Web` repository. Connect your GitHub account if asked.
2. Set **Root Directory** to `fidential-website`. Leave **Framework Preset** on **Other**, with no build command.
3. Click **Deploy**, then add the domain (see Domain settings).

### Publish an update
Make the change, then in GitHub Desktop write a short summary, click **Commit to main**, then **Push origin**. Vercel publishes it in about a minute.

Vercel keeps earlier versions. To undo an update, open the project's **Deployments** tab, find the previous version and choose **Instant Rollback** from its menu.

You can also publish without GitHub, from a terminal opened in `fidential-website/`: run `npx vercel login` once, then `npx vercel --prod` for each update.

### Domain settings
In Vercel, the project's **Settings → Domains** page should show:
- `fidential.shop`: connected to Production.
- `www.fidential.shop`: redirects to `fidential.shop`.

The address without `www` is the main one. The canonical tag, sitemap, link previews and search-engine data all use it. When you add the domain, Vercel suggests the opposite (sending visitors to `www`), so check that the setup above is what you see.

In GoDaddy, the domain's **DNS → DNS Records** need these two records:

| Type | Name | Value |
|---|---|---|
| A | `@` | The A value on Vercel's Domains page (usually `76.76.21.21`) |
| CNAME | `www` | The CNAME value on Vercel's Domains page (looks like `d1d4fc829fe7bc7c.vercel-dns-017.com`) |

Delete any other A or AAAA records named `@`. Leave the NS, SOA, MX, TXT and `_domainconnect` records alone. GoDaddy's **Connect Domain** button is for its own site builder, so don't use it.

### Preview on your computer
Serve the folder instead of double-clicking `index.html`. Browsers block 3D textures on `file://` pages, so the hero falls back to a flat image.
- VS Code: click "Go Live" (Live Server). `.vscode/settings.json` already points it at `fidential-website/`.
- Terminal, in `fidential-website/`: run `npx serve`, then open the address it prints.

Everything the site needs is in the folder, so the preview works offline too, apart from sending reviews and opening WhatsApp.

### Keep the folder on this Mac
The repo is on the Desktop, which iCloud syncs. When the Mac runs low on space, macOS moves files to iCloud, and Live Server or a deploy fails if a file isn't on the Mac. In Finder, right-click the `Fidential-Web` folder and choose **Keep Downloaded**.

Built by IONIK — ionikcollectives.com
