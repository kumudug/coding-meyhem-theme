# Featured Posts Section — Design Spec

**Date:** 2026-05-25  
**Status:** Approved  
**File:** `coding-mayhem-workshop.xml`

---

## Overview

Add a curated "Featured Posts" section to the blog homepage that appears between the hero (latest post) and the regular chronological posts feed. Featured posts are selected by applying a `featured` label in Blogger.

---

## Homepage Page Flow

```
[Hero: latest post]              ← unchanged
[★ Featured: up to 3 posts]      ← new section
[All posts: chronological feed]  ← unchanged (featured posts also appear here)
[Pager]
```

The featured section is **homepage-only**. It does not appear on label, archive, or search pages.

Featured posts also appear in the regular "all posts" feed below — no exclusion logic needed.

---

## Selection Mechanism

Posts are featured by tagging them with the `featured` label in Blogger. The section fetches up to 3 posts with that label via the Blogger JSON API. The user manages featured posts entirely through Blogger's label system — no template changes required to add/remove featured posts.

---

## GML Template Changes

Add a `<div id="featured-section" class="featured-section"></div>` placeholder inside the `b:if cond='data:view.isHomepage'` block, between the closing `</section>` of the hero and the `<b:with>` posts grid:

```xml
<!-- after hero </section> -->
<div id='featured-section' class='featured-section'></div>

<!-- existing b:with posts grid -->
<b:with value='...' var='posts'>
  ...
</b:with>
```

No other GML structure changes required.

---

## JavaScript

### `cmLoadFeatured()`

Called in `DOMContentLoaded` alongside existing init functions.

```js
function cmLoadFeatured() {
  var section = document.getElementById('featured-section');
  if (!section) return;  // not homepage, bail
  var feed = (window.location.origin || '') +
    '/feeds/posts/default/-/featured?alt=json-in-script&max-results=3&callback=cmFeatured';
  var s = document.createElement('script');
  s.src = feed;
  document.body.appendChild(s);
}
```

### `window.cmFeatured(data)` callback

- Reads up to 3 entries from `data.feed.entry`
- For each entry extracts: `url` (from `link[rel=alternate].href`), `title` (`title.$t`), `published date` (`published.$t`), `excerpt` (`summary.$t`, truncated to ~120 chars), `media$thumbnail` url (resized to 720px by replacing the size segment), `category` labels (skipping the `featured` label itself)
- Renders the **1-large + 2-small** layout into `#featured-section`
- **Edge cases:**
  - 0 posts tagged `featured`: section stays empty (no visible output, CSS `display:none` via `:empty`)
  - 1–2 posts tagged: renders however many exist; layout adapts (small-cards column may have 1 item)
  - Feed/network error: callback never fires, section stays empty silently

### Rendering pattern

```
featured-section
  ├── featured-label  ("★ featured")
  └── featured-grid
        ├── featured-big-card    (entry[0]: cover image, tag pill, title, excerpt)
        └── featured-small-cards
              ├── featured-small-card  (entry[1]: thumbnail, title, date)
              └── featured-small-card  (entry[2]: thumbnail, title, date)
```

---

## CSS

Added to `<b:skin>`. Uses only existing CSS variables — no new tokens.

### Structure

```css
/* Section wrapper */
.featured-section {
  padding: 32px 0;
  border-bottom: 1px solid var(--rule);
}
.featured-section:empty { display: none; }

/* Label row */
.featured-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px; letter-spacing: 0.14em; text-transform: uppercase;
  color: var(--muted); margin-bottom: 16px;
  display: flex; align-items: center; gap: 12px;
}
.featured-label::after { content: ''; flex: 1; height: 1px; background: var(--rule); }

/* Desktop grid: 1 large + 2 small */
.featured-grid {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 20px;
  align-items: start;
}

/* Big card (left) */
.featured-big-card { ... }         /* cover img, tag pill, title, excerpt — same tokens as .post-card */

/* Small cards column (right) */
.featured-small-cards {
  display: flex; flex-direction: column; gap: 12px;
}
.featured-small-card { ... }       /* horizontal: thumbnail left, title + date right — same tokens as .adjacent-card */

/* Mobile: stack vertically at ≤900px */
@media (max-width: 900px) {
  .featured-grid { grid-template-columns: 1fr; }
  /* big card goes full-width; small-cards remain stacked as rows beneath */
}
```

---

## Behaviour Summary

| Scenario | Result |
|---|---|
| 3 posts tagged `featured` | Full 1-large + 2-small layout |
| 1–2 posts tagged `featured` | Partial layout (only available cards render) |
| 0 posts tagged `featured` | Section hidden (`:empty`) |
| Non-homepage view | Placeholder div absent; no fetch fires |
| Featured post also in feed | Appears in both sections — no deduplication |

---

## Out of Scope

- Animating the featured section load
- Excluding featured posts from the regular feed
- Showing more than 3 featured posts
- Featured section on non-homepage views
