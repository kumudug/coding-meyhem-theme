# Featured Posts Section Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a curated "Featured Posts" section to the homepage that renders between the hero and the chronological posts feed, populated via Blogger's JSON API from posts tagged `featured`.

**Architecture:** A placeholder `<div id="featured-section">` is added to the GML template (homepage only). On page load, `cmLoadFeatured()` injects a JSONP script tag fetching posts with the `featured` label. The `cmFeatured` callback renders a 1-large-+2-small card layout into the placeholder. CSS handles the two-column desktop grid and single-column mobile stack.

**Tech Stack:** Blogger GML XML template, vanilla JS (ES5, CDATA-safe), CSS custom properties (existing token set)

---

## File Map

| File | Change |
|---|---|
| `coding-mayhem-workshop.xml` | All changes — CSS (line 959), GML placeholder (line 1122), JS functions + boot call (~line 1703 and 1743) |

---

### Task 1: Add CSS for the featured section

**Files:**
- Modify: `coding-mayhem-workshop.xml:959` (insert before `]]></b:skin>`)

- [ ] **Step 1: Insert featured section CSS before the closing `]]></b:skin>` tag (line 960)**

  Find this line (line 959):
  ```
  body#layout .page-outer > .sidebar { display: block; }
  ```

  Add the following block immediately after it (before `  ]]></b:skin>`):

  ```css

  /* ── Featured Posts ──────────────────────────────────── */
  .featured-section { padding: 32px 0; border-bottom: 1px solid var(--rule); }
  .featured-section:empty { display: none; }
  .featured-label {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px; letter-spacing: 0.14em; text-transform: uppercase;
    color: var(--muted); margin-bottom: 16px;
    display: flex; align-items: center; gap: 12px;
  }
  .featured-label::after { content: ''; flex: 1; height: 1px; background: var(--rule); }
  .featured-grid {
    display: grid;
    grid-template-columns: 1.5fr 1fr;
    gap: 20px;
    align-items: start;
  }
  .featured-big-card {
    background: var(--surface);
    border: 1px solid var(--rule);
    border-radius: 14px;
    overflow: hidden;
    display: block;
    transition: transform .15s, box-shadow .15s, border-color .15s;
  }
  .featured-big-card:hover { transform: translateY(-2px); box-shadow: var(--shadow); border-color: var(--accent); }
  .featured-big-card-cover {
    display: block; width: 100%; aspect-ratio: 16 / 9;
    overflow: hidden; background: var(--surface-2);
  }
  .featured-big-card-cover img { width: 100%; height: 100%; object-fit: cover; }
  .featured-big-card-cover .featured-fallback { width: 100%; height: 100%; border-radius: 0; min-height: 160px; }
  .featured-big-card-body { padding: 18px; }
  .featured-big-card-tag {
    display: inline-flex; align-items: center;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px; padding: 3px 8px; border-radius: 6px;
    background: var(--chip-bg); color: var(--chip-ink); margin-bottom: 10px;
  }
  .featured-big-card-title {
    margin: 0 0 10px;
    font-weight: 700; font-size: 20px; line-height: 1.2;
    letter-spacing: -0.015em; color: var(--ink); text-wrap: pretty;
  }
  .featured-big-card-excerpt {
    margin: 0 0 10px; font-size: 14px; line-height: 1.55; color: var(--body);
    display: -webkit-box; -webkit-line-clamp: 3; -webkit-box-orient: vertical; overflow: hidden;
  }
  .featured-big-card-date { font-family: 'JetBrains Mono', monospace; font-size: 11.5px; color: var(--muted); }
  .featured-small-cards { display: flex; flex-direction: column; gap: 12px; }
  .featured-small-card {
    background: var(--surface); border: 1px solid var(--rule);
    border-radius: 12px; display: flex; overflow: hidden;
    transition: transform .15s, border-color .15s;
  }
  .featured-small-card:hover { transform: translateY(-2px); border-color: var(--accent); }
  .featured-small-card-thumb {
    width: 88px; flex-shrink: 0; background: var(--surface-2); overflow: hidden;
  }
  .featured-small-card-thumb img { width: 100%; height: 100%; object-fit: cover; }
  .featured-small-card-thumb .featured-fallback { width: 100%; height: 100%; border-radius: 0; min-height: 80px; }
  .featured-small-card-body { padding: 12px 14px; display: flex; flex-direction: column; justify-content: center; }
  .featured-small-card-title {
    margin: 0 0 6px; font-weight: 700; font-size: 15px;
    line-height: 1.25; color: var(--ink); text-wrap: pretty;
  }
  .featured-small-card-date { font-family: 'JetBrains Mono', monospace; font-size: 11px; color: var(--muted); }
  @media (max-width: 900px) {
    .featured-grid { grid-template-columns: 1fr; }
  }
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add coding-mayhem-workshop.xml
  git commit -m "feat: add featured section CSS"
  ```

---

### Task 2: Add the GML placeholder div

**Files:**
- Modify: `coding-mayhem-workshop.xml:1122` (insert after line 1122)

- [ ] **Step 1: Insert the placeholder between the hero and the posts grid**

  Find this block (lines 1122–1124):
  ```xml
              </b:if>
              <!-- INDEX views (home minus hero, label, archive, search): post cards grid -->
              <b:with value='data:view.isHomepage ? (data:posts filter (post =&gt; post.id != data:posts.first.id)) : data:posts' var='posts'>
  ```

  Replace it with:
  ```xml
              </b:if>
              <!-- Featured posts placeholder (homepage only, populated via JS) -->
              <b:if cond='data:view.isHomepage'>
                <div id='featured-section' class='featured-section'></div>
              </b:if>
              <!-- INDEX views (home minus hero, label, archive, search): post cards grid -->
              <b:with value='data:view.isHomepage ? (data:posts filter (post =&gt; post.id != data:posts.first.id)) : data:posts' var='posts'>
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add coding-mayhem-workshop.xml
  git commit -m "feat: add featured-section placeholder div to GML template"
  ```

---

### Task 3: Add JavaScript functions and wire into boot

**Files:**
- Modify: `coding-mayhem-workshop.xml` — JS block (~line 1703, before `/* Boot */`, and line 1743 inside DOMContentLoaded)

- [ ] **Step 1: Add `cmLoadFeatured` and `window.cmFeatured` before the `/* Boot */` comment**

  Find this line (~line 1703 after Task 2 line shift):
  ```javascript
    /* Boot */
    document.addEventListener('DOMContentLoaded', function(){
  ```

  Insert the following block immediately before it:
  ```javascript
    /* Featured posts */
    function cmLoadFeatured() {
      var section = document.getElementById('featured-section');
      if (!section) return;
      var feed = (window.location.origin || '') +
        '/feeds/posts/default/-/featured?alt=json-in-script&max-results=3&callback=cmFeatured';
      var s = document.createElement('script');
      s.src = feed;
      document.body.appendChild(s);
    }
    window.cmFeatured = function(data) {
      var section = document.getElementById('featured-section');
      if (!section || !data || !data.feed || !data.feed.entry || !data.feed.entry.length) return;
      var entries = data.feed.entry;
      function getUrl(e) {
        for (var i = 0; i < e.link.length; i++) {
          if (e.link[i].rel === 'alternate') return e.link[i].href;
        }
        return '#';
      }
      function getThumb(e) {
        if (e.media$thumbnail && e.media$thumbnail.url) {
          return e.media$thumbnail.url.replace(/\/s\d+(-c)?\//, '/s720/');
        }
        return '';
      }
      function fmtDate(str) {
        return new Date(str).toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' });
      }
      function getLabel(e) {
        if (e.category) {
          for (var i = 0; i < e.category.length; i++) {
            if (e.category[i].term.toLowerCase() !== 'featured') return e.category[i].term;
          }
        }
        return '';
      }
      var html = '<div class="featured-label">★ featured</div><div class="featured-grid">';
      var e0 = entries[0];
      var url0 = getUrl(e0);
      var title0 = (e0.title && e0.title.$t) || '';
      var excerpt0 = (e0.summary && e0.summary.$t) || '';
      if (excerpt0.length > 120) excerpt0 = excerpt0.slice(0, 120).replace(/\s\S+$/, '') + '…';
      var thumb0 = getThumb(e0);
      var label0 = getLabel(e0);
      html += '<a class="featured-big-card" href="' + url0 + '">';
      html += '<div class="featured-big-card-cover">' +
        (thumb0
          ? '<img src="' + thumb0 + '" alt="' + title0.replace(/"/g, '&quot;') + '">'
          : '<div class="featured-fallback">NO COVER</div>') +
        '</div>';
      html += '<div class="featured-big-card-body">';
      if (label0) html += '<span class="featured-big-card-tag">#' + label0 + '</span>';
      html += '<h3 class="featured-big-card-title">' + title0 + '</h3>';
      if (excerpt0) html += '<p class="featured-big-card-excerpt">' + excerpt0 + '</p>';
      html += '<span class="featured-big-card-date">' + fmtDate(e0.published.$t) + '</span>';
      html += '</div></a>';
      if (entries.length > 1) {
        html += '<div class="featured-small-cards">';
        for (var i = 1; i < entries.length; i++) {
          var e = entries[i];
          var url = getUrl(e);
          var title = (e.title && e.title.$t) || '';
          var thumb = getThumb(e);
          html += '<a class="featured-small-card" href="' + url + '">';
          html += '<div class="featured-small-card-thumb">' +
            (thumb
              ? '<img src="' + thumb + '" alt="' + title.replace(/"/g, '&quot;') + '">'
              : '<div class="featured-fallback">NO COVER</div>') +
            '</div>';
          html += '<div class="featured-small-card-body">' +
            '<h3 class="featured-small-card-title">' + title + '</h3>' +
            '<span class="featured-small-card-date">' + fmtDate(e.published.$t) + '</span>' +
            '</div></a>';
        }
        html += '</div>';
      }
      html += '</div>';
      section.innerHTML = html;
    };

  ```

- [ ] **Step 2: Add `cmLoadFeatured()` to the DOMContentLoaded boot block**

  Find this line inside DOMContentLoaded:
  ```javascript
      cmLoadAdjacent();
  ```

  Add `cmLoadFeatured()` immediately after it:
  ```javascript
      cmLoadAdjacent();
      cmLoadFeatured();
  ```

- [ ] **Step 3: Commit**

  ```bash
  git add coding-mayhem-workshop.xml
  git commit -m "feat: add cmLoadFeatured JS and wire into boot"
  ```

---

### Task 4: Upload to Blogger and verify

No automated test runner is available for Blogger XML templates. Verification is by uploading and inspecting the live blog.

**Before starting:** Make sure at least 3 posts on your blog exist to test with.

- [ ] **Step 1: Upload the template**

  1. Go to [https://www.blogger.com](https://www.blogger.com)
  2. Select your blog → **Theme** → click the **▼ arrow next to Customise** → **Restore** → Upload `coding-mayhem-workshop.xml`

- [ ] **Step 2: Verify — 0 featured posts (section hidden)**

  Visit your blog homepage. The featured section div is present in the DOM but empty. Confirm:
  - No `★ featured` label and no cards appear between the hero and the posts grid
  - The page looks identical to before (`:empty` CSS hides the section)
  - Open DevTools → Elements: `<div id="featured-section" class="featured-section"></div>` exists but has no children

- [ ] **Step 3: Tag 1 post as `featured` and verify partial layout**

  In Blogger → Posts, open any post and add the label `featured`. Visit the homepage. Confirm:
  - `★ featured` label appears with a rule line
  - One big card (1.5fr column) renders with cover image, tag pill, title, excerpt, date
  - The right column (small cards) is absent since only 1 entry returned
  - The regular posts grid still renders below

- [ ] **Step 4: Tag 3 posts as `featured` and verify full layout**

  Add `featured` label to two more posts. Visit the homepage. Confirm:
  - `★ featured` label row visible
  - Desktop: big card on the left, two small cards (thumbnail + title + date) stacked on the right
  - All 3 cards are clickable links to the correct posts
  - Featured posts also appear again in the chronological feed below (no deduplication)
  - Hover: cards lift (`translateY(-2px)`) and border turns accent colour

- [ ] **Step 5: Verify mobile layout**

  Resize browser to < 900px (or use DevTools device toolbar). Confirm:
  - `featured-grid` collapses to single column
  - Big card renders full-width
  - Two small cards stack vertically below it as horizontal rows (thumbnail left, text right)

- [ ] **Step 6: Verify non-homepage views are unaffected**

  Visit a single post, a label page, and a search page. Confirm:
  - No `featured-section` div in the DOM on any of these views
  - No network request to `/feeds/posts/default/-/featured` fires (check DevTools → Network)

- [ ] **Step 7: Commit verification note**

  ```bash
  git commit --allow-empty -m "chore: verified featured posts section on live Blogger blog"
  ```
