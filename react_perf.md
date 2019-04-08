---
theme: style.css
verticalSeparator: -v-
highlightTheme: github
revealOptions:
  transition: none
---

<!-- .slide: data-background-color="#673399" -->
<h1 class="title dark-background"><span class="translucent">Advanced React</span> Performance</h1>
<h2 class="subtitle">Jason Lengstorf + Sara Vieira <br>+ Sia Karamalegos</h2>

---

## Meet today’s teachers

- Sara Vieira ([@NikkitaFTW](https://twitter.com/NikkitaFTW))
- Jason Lengstorf ([@jlengstorf](https://twitter.com/jlengstorf))

---


## Coding is more fun with friends

- 👋 Introduce yourself to your neighbors <!-- .element: class="fragment" -->
- 👯‍♀️ Pair programming is a great option! <!-- .element: class="fragment" -->
- 💬 Ask lots of questions <!-- .element: class="fragment" -->

Note: Suggest pair programming and give them an opportunity to change seats.

---

## Let’s look at our app

https://git.io/advanced-react-perf

---

## Why’s my app so slow?

Let’s debug it! <!-- .element: class="fragment" -->

---

## Step 0: Understand the problems

- Analyze the app assets <!-- .element: class="fragment" -->
- Look at performance audit reports <!-- .element: class="fragment" -->
- Identify opportunities to improve <!-- .element: class="fragment" -->
- Prioritize the list using an impact vs. effort matrix <!-- .element: class="fragment" -->
- Make a plan of action! <!-- .element: class="fragment" -->

---

## Agenda

1. Low-hanging fruit
2. Code splitting
3. Refactoring dependencies
4. Latency and font loading
5. Caching with service workers
6. Perceived performance and lazy loading
7. Optimizing images

---

# Low-hanging fruit

Note: Run the bundle analyzer.<br>
Turn on production builds for webpack.<br>
Add terser.<br>
Extract CSS.<br>
Minify CSS.<br>
Set browser targets for Babel (@babel/preset-env).<br>
Enable gzip compression.<br>

---

# Code splitting

Note: Lazy load large deps (the Prettier + plugins setup)<br>
Lazy load routes and large components (React.lazy + Suspense)<br>
Add chunk names for easier debugging<br>

---

# Refactoring dependencies

Note: Move Prettier to admin dashboard vs. loading on the client side.<br>
Refactor Moment.js to use date-fns (see next slide).<br>
To mention: lodash vs. lodash-es (see next slide).<br>

---

## Module Imports

```javascript
// Big
import _ from 'lodash';
_.isEmpty({});

// Big
import {isEmpty} from 'lodash';
isEmpty({});

// Little
import isEmpty from 'lodash/isEmpty';
isEmpty({})

// Big
import moment from 'moment';

// Little
import addMinutes from 'date-fns/addMinutes';
```

<small>Use Moment? Try [date-fns](https://date-fns.org/) instead.</small>

---

# Latency and resource hints

Note: Optimize font loading with resource hints.<br>
Switch from CSS `@import` to HTML `<link />` for Google fonts.<br>
Preconnect fonts.gstatic.com<br>
Preload local fonts that are definitely used.<br>
Add font-display: swap for local fonts.<br>

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

# Caching with service workers

Note: Set up workbox.<br>
Load the SW in index.html.<br>
Now, all the bundles will be preloaded, and that’s ~1.5MB.<br>
Ignore vendor bundles.<br>
Set up runtime caching for vendor bundles.<br>
Add runtime caching for Google Fonts.<br>

---

<img style="border:none;box-shadow:none;" src="./images/Workbox-Logo-Grey.svg" />

<small>[Docs](https://developers.google.com/web/tools/workbox/)</small>

---

# Perceived performance

Note: Show things as loaded instead of blocking on the slowest thing.<br>
Refactor the dashboard to load metrics inside the chart.<br>
Use Suspense to load the chart.<br>

---

# Optimizing images

---

## Want a mini-course in responsive images?

File formats, `srcset`'s, and `<picture>`'s, oh my!

https://siakaramalegos.github.io/responsive-images-slides/#/

---

# Script execution costs? (third-party scripts)

---

<!-- .slide: data-background-color="#673399" -->
<h1 class="title dark-background">Thanks!</h1>
<!-- Slides, resources, and more at <a href="https://bit.ly/siaspeaks" class="dark-background">bit.ly/siaspeaks</a> -->
