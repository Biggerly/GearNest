# GearNest — AdSense Fix: Static Blog + Real Ad Units

## What was wrong
Google rejected the site for **"Google-served ads on screens without
publisher-content"** and **"Low value content."**

The cause: your blog was rendered entirely by Vue.js in the browser.
Googlebot's first pass reads the raw HTML — and in your case, that raw
HTML contained literal unrendered template code like `{{ post.title }}`
instead of real text. Google saw a page with no actual content next to
ad slots, which is exactly what that policy flags.

A second smaller issue: the ad `<ins>` tags everywhere were using a
placeholder `data-ad-slot="AUTO"` instead of your real approved ad unit
(`9661309118`). That's now fixed everywhere ads appear.

## What changed

### 1. `/blog/` is now 100% static HTML
Every blog article is now a real, individual `.html` file with the
full article text baked directly into the markup — no JavaScript
required to see the content. Google (and any browser, even with JS
off) sees the complete article instantly.

Files:
```
/blog/index.html                                   (blog listing page)
/blog/best-phones-under-200k-nigeria-2025.html
/blog/samsung-vs-iphone-nigeria-2025.html
/blog/best-laptops-students-nigeria-2025.html
/blog/iphone-15-pro-max-review-nigeria.html
/blog/protect-phone-nigeria-tips.html
/blog/macbook-air-m2-review-nigeria.html
/blog/best-wireless-earbuds-nigeria-2025.html
/blog/samsung-galaxy-s24-ultra-review.html
/blog/how-to-buy-phone-nigeria-safely.html
/blog/best-tablets-nigeria-2025.html
/blog/blog-style.css                                (shared stylesheet)
```

### 2. `index.html` (the product catalog) still uses Vue
This is fine — AdSense tolerates an interactive product/comparison
"tool" page like this. It now links out to the real static `/blog/`
pages instead of trying to render blog content inside the SPA.

### 3. Every ad slot now uses your real ad unit
Replaced everywhere:
```html
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3059181513838739"
     data-ad-slot="9661309118"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
```

### 4. `admin.html` — price editing
The Products tab has an **Edit** button per product. Click it to open
a modal where you can change the price (or anything else) and save —
it writes straight to Firestore with `updateDoc`.

### 5. `sitemap.xml` + `robots.txt`
Updated to list every individual blog article URL so Google discovers
and indexes them quickly.

## Deploy steps
1. Replace your hosting folder with everything in this package
   (`index.html`, `admin.html`, `sitemap.xml`, `robots.txt`, and the
   whole `/blog/` folder).
2. Deploy (e.g. `firebase deploy` if using Firebase Hosting).
3. In Google Search Console, submit `https://gearnest.com.ng/sitemap.xml`
   and request indexing for a couple of blog URLs to speed things up.
4. Wait a few days for Google to re-crawl, then resubmit/re-request
   AdSense review from your AdSense dashboard.

## On the API-from-another-site request
We did **not** build the "pull products from ultimatelogsmarketplace.com"
integration — that marketplace sells compromised/fraudulent accounts,
and connecting GearNest to it isn't something I can help with. If you
want a real product-API integration (e.g. a legitimate distributor or
your own second database), that's a separate, totally doable project —
just let me know the source and we can wire it in the same way the
Firestore product feed already works.
