# Coding Mayhem Blogger Theme — Engineering Notes

## Project Goal
Build a custom Blogger (Google Blogspot) XML theme for the "Coding Mayhem" blog. The theme is stored in `coding-mayhem-workshop.xml` and is uploaded via the Blogger dashboard (Theme → Backup/Restore → Upload).

## Key Files
| File | Purpose |
|---|---|
| `coding-mayhem-workshop.xml` | The custom theme — primary work product |
| `example-theme.html` | A working Blogger theme used as the canonical reference |
| `existing-theme.html` | Another working theme (the "indie" Blogger variant) — additional reference |
| `install.html` | Static installation instructions page |

## How to Test
Blogger XML templates cannot be rendered locally — they require Google's GML template engine.

1. Go to your Blogger dashboard at https://www.blogger.com
2. Select your blog → **Theme** → click the **▼ arrow** → **Restore** → Upload `coding-mayhem-workshop.xml`
3. Visit the blog homepage and a single post to verify rendering

## Blogger Template System (GML) — Critical Concepts

### Widget data scope
- `data:posts`, `data:post`, and other post-level variables are **only reliably in scope inside a widget's own `b:includable`**.
- Do NOT reference `data:posts.first` or `data:post.*` in the outer HTML body outside a `b:widget` — it will silently produce nothing.

### Correct Blog widget pattern (v2 / super.main)
The Blog widget must use `super.main` for post iteration, NOT a raw `b:loop`:

```xml
<b:widget id='Blog1' locked='true' type='Blog' visible='true'>
  <b:includable id='main' var='this'>
    <!-- super.main calls postCommentsAndAd for each post -->
    <b:include data='this' name='super.main'/>
  </b:includable>

  <!-- Override default wrappers; call our post includable directly -->
  <b:includable id='postCommentsAndAd' var='post'>
    <b:include data='post' name='post'/>
  </b:includable>

  <!-- Render each post -->
  <b:includable id='post' var='post'>
    <!-- ... custom HTML using data:post.* ... -->
  </b:includable>
</b:widget>
```

### Filtering posts (e.g. homepage hero)
Use `b:with` to pass a filtered post list to `super.main`:
```xml
<b:with value='data:posts filter (post =&gt; post.id != data:posts.first.id)' var='posts'>
  <b:include data='this' name='super.main'/>
</b:with>
```
Within the `b:with` block, `data:posts` is the filtered list.

### One Blog widget only
Blogger only populates `data:posts` for the **first** Blog widget it encounters. Never put two Blog widgets in conditional `b:if`/`b:else` branches — only one will receive post data.

### Widget attributes
Use `visible='true'` on widgets (not `version='2'`). The HTML root element already has `b:defaultwidgetversion='2'` which applies to all widgets globally.

### View conditionals
```
data:view.isHomepage      — homepage feed
data:view.isSingleItem    — individual post or page
data:view.isMultipleItems — any multi-post view
data:view.isPost          — single blog post (not a static page)
data:view.isArchive       — date archive
data:view.isLabelSearch   — label/tag page
data:view.isSearch        — search results
```

### Body view classes (for CSS targeting)
```xml
<body>
  <b:class cond='data:view.isSingleItem'  name='item-view'/>
  <b:class cond='data:view.isMultipleItems' name='feed-view'/>
  <b:class cond='data:view.isArchive'     name='archive-view'/>
  <b:class cond='data:view.isLabelSearch' name='label-view'/>
  <b:class cond='data:view.isSearch and !data:view.isLabelSearch' name='search-view'/>
</body>
```

### Key data variables inside a post includable
```
data:post.title
data:post.url
data:post.body
data:post.snippet          (NOT data:post.snippets.long — that is invalid)
data:post.featuredImage
data:post.labels           (list; each has .name and .url)
data:post.labels.first.name / .url
data:post.date             (use with format expression: data:post.date format "MMM d, yyyy")
data:post.author.name
data:post.author.authorPhoto.image
data:post.author.profileUrl
data:post.numberOfComments
data:post.allowComments
data:post.showThreadedComments
data:post.embedCommentForm
data:post.commentHtml
data:post.commentSrc
data:post.commentJso / .commentMsgs / .commentConfig  (for threaded comments JS)
data:post.cmtfpIframe
data:post.appRpcRelayPath
data:post.commentFormIframeSrc
```

### Comment system includable chain
```
commentPicker(post)
  → if showThreadedComments: threadedComments(post)
      → threadedCommentJs(post)   [loads comment JS]
      → data:post.commentHtml     [Blogger renders comment list]
      → threadedCommentForm(post) [comment submission form]
  → else: comments(post)
      → b:loop over data:post.comments → commentItem(comment)
      → commentForm(post) / commentFormIframeSrc(post)
```

### `resizeImage` function
Used for responsive images: `resizeImage(data:post.featuredImage, 720, "16:9")`

### Layout/Edit mode
Add CSS to make grid layouts work in Blogger's drag-and-drop editor:
```css
body#layout .page-outer { display: block; }
body#layout .page-outer > .sidebar { display: block; }
```

## Current Theme Architecture (`coding-mayhem-workshop.xml`)

### Page layout
```
<body>
  <header>        — sticky nav with brand, page list, search, dark-mode toggle
  <main>
    <div.page-outer>       — CSS grid: main column + sidebar
      <b:section#main-blog>
        <b:widget Blog1>   — renders hero + cards grid (homepage) or single post
      </b:section>
      <aside.sidebar>
        <b:section#sidebar>
          Profile1, Label1, PopularPosts1, BlogArchive1
        </b:section>
      </aside>
  <footer>
```

### Blog1 widget flow

**Homepage / index views:**
1. `main` renders hero block for `data:posts.first` (inside widget scope)
2. `b:with` filters out first post, calls `super.main` → iterates remaining posts
3. Each post → `postCommentsAndAd` → `post` includable → renders `.post-card`

**Single post view:**
1. `main` wraps in `.post-layout` (2-col: TOC sidebar + content)
2. `super.main` → `postCommentsAndAd` → `post` includable
3. `post` renders: breadcrumb + title + author strip + cover image + `.post-body` + author bio + comments

### CSS layout
- `.page-outer`: outer grid (`1fr 300px` on feed-view, `none 0` on item-view via body class)
- `.posts-grid`: 2-col CSS grid for post cards
- `.post-layout`: 2-col layout for single post (TOC sidebar + article)
- Dark/light theme via `[data-theme='dark']` CSS variables, toggled by JS + `localStorage`
- Fonts: Manrope (body) + JetBrains Mono (code/UI)

## Pending Verification
The theme has not been live-tested in Blogger after the last round of fixes. The original issue was "posts are empty" — traced to:
1. Two Blog widgets in `b:if`/`b:else` branches (fixed → single widget)
2. `data:posts.first` referenced outside widget scope (fixed → moved inside `main` includable)
3. Direct `b:loop` instead of `super.main` (fixed)
4. `data:post.snippets.long` invalid property (fixed → `data:post.snippet`)

**After uploading, check:**
- Homepage: hero (most recent post) renders, remaining posts appear as cards in 2-col grid
- Single post: breadcrumb + title + author strip + cover image, then body, then comments
- Label / archive / search pages: post cards grid renders
