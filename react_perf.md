---
theme: style.css
verticalSeparator: -v-
highlightTheme: github
---

<!-- .slide: data-background="./images/hero_bg.jpg" -->
<h1 class="title dark-background"><span class="translucent">React App</span> Performance Tuning</h1>
<h2 class="subtitle">Jason Lengstorf + Sara Vieira <br>+ Sia Karamalegos</h2>

---

## Meet the speakers

---


## introduce yourself to your neighbors

Note: Suggest pair programming and give them an opportunity to change seats.

---

## Let’s look at our app

https://github.com/jlengstorf/advanced-react-perf?

Notes:

Steps in this workshop:

1. Assessing an existing React app (60m)
  Install `webpack-bundle-analyzer` to see what’s in the bundle
  DevTools (lighthouse) to see how it performs now — mention webpagetest.org
  Review webpack config (e.g. production bundles, analyzer, etc.)
  Add terser
  Show difference in React/React DOM bundle sizes after turning on production mode
  CSS minification — have configured but commented out
  Gzip/brotli — have configured but commented out
2. Diagnosing performance problems & prioritizing highest-impact solution (15m)
  Workshop w/the class. Outcome needs to match the list of fixes we want to implement. :)
  Identify and walk through low-hanging fruit
3. Lazy loading resources & components with React.lazy and Suspense (40m)
  Set up the Router to use React.lazy and Suspense
  Use React.lazy to load Code Mirror
  Show how this creates code splitting and explain what that means
4. Leveraging service workers for performance (30m)
  https://developers.google.com/web/tools/workbox/guides/codelabs/webpack
5. Seamlessly preloading and prefetching assets (35m)
  Let’s preload fonts
  Set up preloading for Dank Mono files
  Font-display: swap
  Move Google Fonts to HTML vs. @import
  Set up preconnect for Google Fonts (Montserrat)
  Discuss the trade-offs of using local Google Fonts vs. hosted fonts (e.g. cache)
  Don’t use icon fonts; use SVGs (react-icons)
6. The future of fonts (10m)
  Explain what subfont is; don’t actually do it
  You can do this in Gatsby or locally
  Talk about font variants
7. Automatically optimizing images (20m)
  We can add svgo https://github.com/rpominov/svgo-loader
  https://www.npmjs.com/package/imagemin-webpack-plugin
8. Mitigating the performance impact of third-party scripts (45m)
  Use request blocking to show how big an impact a given script
  Performance tab, click the “reload” button to get an initial load
  Block reqeusts to domain(s) using devtools
  Click the “reload” perf button again to get a new measurement
  You can now compare between the two loads to see the difference
  Show the secret menu (press shift six times in the “experiments” settings tab)
  Add a Twitter embed; break the script load to _not_ be async
  Refactor to be async; show before and after on the network timeline
  Can we find that site that couldn’t get GDPR compliance and shipped a WAY better website to the EU?
  Show how to use the “show third-party badges” thing in Chrome DevTools
9. Using psychology to make an app feel faster than it actually is (30m)
  Fancy loaders make people think things are faster
  Leave blank (no loader)
  Add a loader
  Staged loading (don’t block the render)

Things we didn’t promise that should probably be solved:
Refactor to use date-fns instead of Moment to show how much smaller the bundle can be
Add lodash to the chart reducer to show how to make sure we’re only including what we actually use



---

## Why's my app so slow???

---

## Contents

1. Lazy-loading resource and components
2. Caching with service workers
3. Latency and resource hints
4. Optimizing images
5. Script execution costs
6. Perceived performance

---

# Lazy-loading resource and components

---

# Caching with service workers

---

# Latency and resource hints

---
<img src="./images/resource-hints.jpg_large" alt="Resource hints cheatsheet find pdf at https://storage.googleapis.com/resource-hints/resource-hints-cheatsheet.pdf" />

<small>https://twitter.com/addyosmani/status/743571393174872064?lang=en</small>

Note: pdf version of this is in the replies to this tweet

---

## Latency Case Study: Fonts

```css
@import url('https://fonts.googleapis.com/css?family=Open+Sans|Muli');

h1 {
  font-family: 'Open Sans', sans-serif;
}

p {
  font-family: 'Muli', sans-serif;
}
```

---

## Loading Google Fonts from CSS

<img src="./images/webfonts_css.png" alt="Google fonts load waterfall showing wasted time from CSS">

---

## Loading Google Fonts from HTML

```html
<link href="https://fonts.googleapis.com/css?family=Muli:400"
      rel="stylesheet">
```

<img src="./images/webfonts_before.png" alt="Google fonts load waterfall showing wasted latency time">

---

## Google Fonts with preconnect!

```html
<link rel="preconnect" href="https://fonts.gstatic.com/" crossorigin>
<link href="https://fonts.googleapis.com/css?family=Muli:400"
      rel="stylesheet">
```

<img src="./images/webfonts_preconnect.png" alt="Google fonts load waterfall showing preconnect">

---

## Webfonts

<ul class="plus-minus">
  <li class="plus">Hosted on fast and reliable CDNs</li>
  <li class="plus">Can provide optimized variants based on user's browser</li>
  <li class="plus">Opportunity for shared caching on popular fonts</li>
  <li class="minus">Minumum of 2 separate requests</li>
  <li class="minus">Can't use resource hints on the font file</li>
  <li class="minus">Doesn't take advantage of HTTP2 multiplexing</li>
  <li class="minus">No control over FOUT or FOIT</li>
</ul>

---

## Self-hosted fonts

```html
<link as="font" type="font/woff2"
  href="./fonts/muli-v12-latin-regular.woff2" crossorigin>

<link as="font" type="font/woff2"
  href="./fonts/muli-v12-latin-700.woff2" crossorigin>
```
<img src="./images/no-preload.png" alt="Self-hosted waterfall showing no preload">

Note: This alone does not fix perf problem.

---

## Preloading self-hosted fonts

```html
<link rel="preload" as="font" type="font/woff2"
  href="./fonts/muli-v12-latin-regular.woff2" crossorigin>

<link rel="preload" as="font" type="font/woff2"
  href="./fonts/muli-v12-latin-700.woff2" crossorigin>
```

<img src="./images/font_preload.png" alt="Self-hosted waterfall showing preload">

<small>Note that `preload` loads a resource whether used or not. Only preload resources that are needed on a particular page. Don't self-host popular webfonts like Open Sans or Roboto (sabotages caching).</small>

Note: `rel="preload"` tells the browser to declaratively fetch the resource but not “execute” it (our CSS will queue usage). `as="font"` tells the browser what it will be downloading so that it can set an appropriate priority. Without it, the browser would set a default low priority. `type="font/woff2` tells the browser the file type so that it only downloads the resource if it supports that file type. `crossorigin` is required because fonts are fetched using anonymous mode CORS.

---

# Optimizing images

---

# Script execution costs

---

# Perceived performance

---

<!-- .slide: data-background="./images/hero_bg.jpg" -->
<h1 class="title dark-background">Thanks!</h1>
Slides, resources, and more at <a href="https://bit.ly/siaspeaks" class="dark-background">bit.ly/siaspeaks</a>

