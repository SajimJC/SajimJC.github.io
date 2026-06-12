---
title: "Why a MkDocs-Style TOC Is Hard in Academic Pages"
date: 2026-06-12
permalink: /posts/2026/06/why-mkdocs-style-toc-is-hard-in-academicpages/
tags:
  - academicpages
  - jekyll
  - toc
  - mkdocs
  - minimal-mistakes
  - frontend
toc: false
---

This post records a problem that looked simple, but turned out to be structurally difficult:

- I wanted a **MkDocs-style right-side headline directory** on blog pages.
- The site is built on **Academic Pages**, which itself is derived from **Minimal Mistakes**.
- In practice, the built-in TOC path, the post layout, the old float-based grid, and the responsive breakpoints did **not** line up cleanly.

The result was not "one missing CSS rule."

The result was a repeated conflict between:

- Markdown-stage TOC generation
- layout-stage HTML rendering
- right sidebar float behavior
- sticky positioning
- responsive breakpoints
- duplicated page title vs body `# H1`

This article is not a final "perfect fix" post.

It is a **traceable engineering note** on why this problem is hard, why the official path is incomplete, and what I learned from trying to force a documentation-style TOC into an academic blog template.

## The Original Goal

The target was visually simple:

- On wide screens, show a **right-side TOC** similar to MkDocs.
- On narrow screens, either keep it usable or gracefully move it above content.
- Let the TOC reflect `## / ### / ####` headings in the article body.

In plain language, I wanted something conceptually close to:

```html
<article class="page">
  <header>...</header>
  <aside class="sidebar__right">
    <nav class="toc">
      <ul class="toc__menu">
        <li><a href="#section-a">Section A</a></li>
        <li><a href="#section-b">Section B</a></li>
      </ul>
    </nav>
  </aside>
  <section class="page__content">
    ...
  </section>
</article>
```

At first glance, this looks trivial.

In reality, the difficulty came from **where the TOC is generated**, **when it is generated**, and **which layout layer controls the right column**.

## What Academic Pages Already Has

Academic Pages already ships with a TOC include:

```liquid
<aside class="sidebar__right">
<nav class="toc" markdown="1">
<header><h4 class="nav__title"><i class="fa fa-{{ include.icon | default: 'file-text' }}"></i> {{ include.title | default: site.data.ui-text[site.locale].toc_label }}</h4></header>
*  Auto generated table of contents
{:toc .toc__menu}
</nav>
</aside>
```

That snippet is important because it reveals the first hidden assumption:

- it expects **Kramdown** to process `{:toc .toc__menu}`
- that means it works naturally when used inside a **Markdown page**
- it does **not** automatically mean it will work when inserted directly into an HTML layout

That distinction turned out to be the first real fault line.

## Why the First Attempt Failed

The first idea was the obvious one:

- turn on `toc: true` by default for posts
- insert the TOC include into `single.html`
- let the blog posts automatically render a right-side directory

Conceptually, that looked like this:

```liquid
<div id="main" role="main">
  {% include sidebar.html %}
  {% if page.toc %}
    {% include toc %}
  {% endif %}

  <article class="page">
    ...
  </article>
</div>
```

But once the Markdown-style TOC include was injected into the HTML layout, the rendered page exposed the raw marker instead of a real directory.

The failure mode was basically:

```html
<nav class="toc">
  <header><h4 class="nav__title">On This Page</h4></header>
  * Auto generated table of contents
  {:toc .toc__menu}
</nav>
```

![Raw TOC marker rendered directly instead of becoming a real table of contents](/files/posts/why-mkdocs-style-toc-is-hard-in-academicpages/raw-toc-marker-rendering.png)

*The screenshot above captures the first hard failure: the page shows the literal `{:toc .toc__menu}` marker instead of a generated TOC. This is the clearest evidence that the rendering-stage assumption was wrong.*

That immediately shows the real problem:

- the TOC marker belongs to the **Markdown rendering stage**
- `single.html` is part of the **layout stage**
- moving the include into the layout broke the assumption under which `{:toc}` was supposed to be expanded

So the first bug was not visual.

It was a **rendering pipeline mismatch**.

## The Second Attempt: Move Toward Minimal Mistakes

At that point the more serious comparison was with Minimal Mistakes, the upstream theme lineage behind many Academic Pages structures.

The upstream idea is closer to:

```liquid
{% if page.toc %}
  <aside class="sidebar__right {% if page.toc_sticky %}sticky{% endif %}">
    <nav class="toc">
      <header>
        <h4 class="nav__title">
          <i class="fa fa-{{ page.toc_icon | default: 'list' }}"></i>
          {{ page.toc_label | default: "On This Page" }}
        </h4>
      </header>
      {% include toc.html sanitize=true html=content h_min=2 h_max=4 class="toc__menu" %}
    </nav>
  </aside>
{% endif %}
```

This is a very different philosophy.

Instead of relying on a Markdown marker such as `{:toc}`, it tries to generate the TOC from the already-rendered `content`.

That is structurally more appropriate for a layout-level TOC.

So I moved toward a custom `toc.html` include that scans the rendered article HTML and extracts headings.

The simplified implementation looked like this:

```liquid
{% assign h_min = include.h_min | default: 1 | plus: 0 %}
{% assign h_max = include.h_max | default: 6 | plus: 0 %}
{% assign headings = include.html | split: '<h' %}

{% capture toc_items %}
{% for heading in headings %}
  {% assign level = heading | slice: 0, 1 | plus: 0 %}
  {% if level >= h_min and level <= h_max %}
    {% capture close_tag %}</h{{ level }}>{% endcapture %}
    {% assign heading_fragment = heading | split: close_tag | first %}
    {% capture heading_html %}<h{{ heading_fragment }}{{ close_tag }}{% endcapture %}
    {% assign heading_id = "" %}
    {% if heading_html contains 'id="' %}
      {% assign heading_id = heading_html | split: 'id="' | last | split: '"' | first %}
    {% endif %}
    {% assign heading_text = heading_html | strip_html | strip %}
    {% if heading_id != "" and heading_text != "" %}
      <li class="toc__item toc__item--level-{{ level }}"><a href="#{{ heading_id }}">{{ heading_text }}</a></li>
    {% endif %}
  {% endif %}
{% endfor %}
{% endcapture %}

{% assign toc_items = toc_items | strip %}
{% if toc_items != "" %}
<ul class="{{ include.class | default: 'toc__menu' }}">
  {{ toc_items }}
</ul>
{% endif %}
```

This fixed one problem:

- the page no longer displayed raw `{:toc}` markers

But it did **not** solve the full layout problem.

## Why the Right Sidebar Still Broke

Once the TOC became a real HTML block, a second class of bugs appeared:

- sometimes the TOC floated correctly on the right
- sometimes it pushed the article downward
- sometimes it appeared above the content instead of beside it
- sometimes it looked correct on one post and wrong on another window size

The key reason was not TOC extraction anymore.

The key reason was **layout placement plus old float logic**.

Academic Pages still uses a float-oriented responsive structure. The relevant right sidebar rule looked like this:

```scss
.sidebar__right {
  margin-bottom: 1em;

  @include breakpoint($large) {
    position: relative;
    float: right;
    width: $right-sidebar-width-narrow;
    margin-left: span(0.5 of 12);
    z-index: 10;
  }

  @include breakpoint($x-large) {
    width: $right-sidebar-width;
  }

  .toc {
    @include breakpoint($large) {
      position: sticky;
      top: calc(#{$masthead-height} + 1rem);
      max-height: calc(100vh - #{$masthead-height} - 2rem);
      overflow-y: auto;
    }
  }
}
```

That means the TOC behavior is tightly coupled to:

- `float: right`
- the `large` breakpoint
- the width rules defined by the theme
- the placement of the TOC block relative to `.page` and `.page__inner-wrap`

If the TOC is inserted at the wrong level, even a correct TOC can still destroy the visual layout.

For example, this kind of placement is risky:

```liquid
<div id="main">
  {% include sidebar.html %}
  <aside class="sidebar__right">...</aside>
  <article class="page">...</article>
</div>
```

because the floating sidebar can interfere with the article as a sibling block.

By contrast, the safer structure is closer to:

```liquid
<article class="page">
  <div class="page__inner-wrap">
    <header>...</header>
    <aside class="sidebar__right">...</aside>
    <section class="page__content">...</section>
  </div>
</article>
```

![A wide-screen layout where the TOC block is effectively treated as a top panel and pushes the article body downward](/files/posts/why-mkdocs-style-toc-is-hard-in-academicpages/toc-pushes-content-down-wide.png)

*This screenshot shows the second class of failure. The TOC is no longer raw text, but the layout placement is wrong enough that it behaves like a large block above the article instead of a clean right-side rail.*

That difference sounds minor, but visually it is not minor at all.

## Why One Screenshot Looked Fine and Another Looked Wrong

At one point, two posts appeared to behave differently.

The natural suspicion was:

- maybe one post had different Markdown
- maybe one post had different front matter
- maybe one post had a special layout

But the real reason was much simpler:

- the right-side TOC only switched into the floated/sticky mode above a specific responsive breakpoint
- below that width, it behaved like a regular block and dropped above content

The theme values were:

```scss
$small       : 600px !default;
$medium      : 768px !default;
$medium-wide : 900px !default;
$large       : 925px !default;
$x-large     : 1280px !default;
```

So the behavior was effectively:

- narrower window: TOC becomes a block above content
- wider window: TOC floats right

The problem is that "above content" is not automatically a bug from the template's point of view, but it is a bug relative to the MkDocs-style experience I was trying to achieve.

The narrow-screen case looked like this:

![A narrower viewport where the TOC stacks above the article body instead of staying in a right-side rail](/files/posts/why-mkdocs-style-toc-is-hard-in-academicpages/toc-stacks-above-content-narrow.png)

*In this state, the TOC is technically present and technically usable, but it has already stopped behaving like a documentation-side navigation panel.*

On a wider viewport, a different failure could appear:

![A wide-screen layout with an obvious empty right-side gap, showing that the responsive rail logic and content area are not truly aligned](/files/posts/why-mkdocs-style-toc-is-hard-in-academicpages/toc-gap-right-rail-wide.png)

*This case is especially instructive because it looks "almost correct" at first glance. But the red-marked empty zone makes the real problem visible: the page has reserved right-side space in a way that does not cleanly correspond to a stable TOC rail.*

I later pushed the TOC threshold down toward `768px`:

```scss
.sidebar__right {
  @include breakpoint($medium) {
    position: relative;
    float: right;
    width: $right-sidebar-width-narrow;
    margin-left: span(0.5 of 12);
    z-index: 10;
  }

  .toc {
    @include breakpoint($medium) {
      position: sticky;
      top: calc(#{$masthead-height} + 1rem);
      max-height: calc(100vh - #{$masthead-height} - 2rem);
      overflow-y: auto;
    }
  }
}
```

This made the behavior **more MkDocs-like**, but it still did not fully solve the deeper mismatch.

It only moved the threshold at which the mismatch becomes visible.

## A Separate Problem: Duplicated H1

There was also a separate issue that repeatedly made the page look "more broken" than it actually was.

In Academic Pages, the layout already prints the page title:

```liquid
{% if page.title %}
  <h1 class="page__title" itemprop="headline">
    {{ page.title | markdownify | remove: "<p>" | remove: "</p>" }}
  </h1>
{% endif %}
```

But if the Markdown body also begins with its own first-level title:

```markdown
# Layered Orchestration in OpenCode: Routing Models, Tasks, and Human Control
```

then the page shows two visible H1-level titles:

- one from layout
- one from body

The visual effect was easy to misread as a TOC positioning problem, but it was actually a different bug layered on top of the TOC experiment:

![A page state where the duplicated title makes the whole layout look more broken than the TOC bug alone would suggest](/files/posts/why-mkdocs-style-toc-is-hard-in-academicpages/duplicated-title-overlap.png)

*This is a good example of why debugging by screenshot alone can be misleading. The TOC may be one problem, but a duplicated H1 can amplify the visual confusion and make the page look structurally worse than it is.*

That is not the TOC bug, but it makes TOC experiments much harder to judge visually.

The cleaner pattern for this site is:

```markdown
---
title: "Layered Orchestration in OpenCode: Routing Models, Tasks, and Human Control"
date: 2026-06-12
---

## Core Point and Introduction
...
```

In other words:

- keep the title in front matter
- let the layout render the visible page title
- start the body from `##`

## Why the Official Path Still Feels Incomplete

After looking back at the template lineage and related discussions, my conclusion is:

- Academic Pages has **pieces** of TOC support
- Minimal Mistakes has a **more layout-oriented direction**
- but neither gives a clean "just add right-side MkDocs TOC to all posts" experience inside a heavily customized Academic Pages fork

That is why this problem feels much harder than expected.

It is not only about:

- CSS
- Liquid
- Markdown
- or one wrong `include`

It is about the friction between two template philosophies:

- **academic blog / profile template**
- **documentation-style navigation template**

They overlap, but they are not the same product.

## What I Learned

My practical takeaway is simple.

If the real goal is **true documentation-site behavior**, then trying to retrofit it into an Academic Pages post layout can easily become a disproportionate amount of work.

The problem is hard because multiple assumptions were never designed together:

```text
Markdown TOC marker
    +
layout-level TOC rendering
    +
old float-based sidebars
    +
post title already rendered by layout
    +
responsive breakpoint shifts
    +
sticky right rail expectations
```

Any one of these is manageable.

All of them together are what create the friction.

So my current view is:

- if I only need a **usable** post directory, I can keep a modest version
- if I want a **real MkDocs experience**, I should seriously consider separating blog pages from document pages rather than forcing one old template to be both

That is the honest conclusion from this round.

## Notes for Future Revision

This post is intentionally written as a **reviewable draft**.

Planned follow-up items:

- add one diagram showing the rendering pipeline mismatch
- add one diagram showing the responsive breakpoint transition
- add links to the exact local files that were modified during the experiment

The image folder for this article is:

```text
files/posts/why-mkdocs-style-toc-is-hard-in-academicpages/
```

I will revise the article after screenshots and diagrams are added.
