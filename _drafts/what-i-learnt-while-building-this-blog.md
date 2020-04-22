---
layout: post
author: HackingPheasant
title: What I learnt while building this blog 
categories: [blog]
tags: [programming, web development, Problems with Solutions]
---

I've tried many times to build a blog, this current iteration so far has lasted the longest, it is also the first time I decided to build a theme for it instead of using something prebuilt.<br>
I encountered my far share of issues (or just lack of knowledge on how to use a certain feature) and new features that where pretty neat and useful (while trying and still implementing [my blogs theme](https://github.com/HackingPheasant/jekyll-theme-pheasants-nest)) that I thought it would be a good idea to document them for future [reference](https://xkcd.com/979/). With this short preamble out of the way lets get into it!

## Using SVG \<use>
When looking online on how others used SVG's on their website I came to realise the [common](https://svgontheweb.com/#implementation) [ways](https://css-tricks.com/using-svg/) that appeared, seemed pretty consistent across a majority of websites. And in my honest opinion, all the options provided seemed rather lackluster to me, as they all had different cons and gotchas, with very few pros.<br> 

Granted the majority of resources you come across when searching, are anywhere (on average) 5 to 10 years old and starting to show their age. Frustrated, I did what I usually do in this situation and search github to see how other projects ended up handling the situation, when I came across [this](https://github.com/jekyll/minima/blob/master/_includes/footer.html#L10-L12). To my amazement, coloring the SVG was as simple as `color: #f66a0a;` and it was orange.
 
A quick search later I end up on the useful Mozilla developer docs to see the overall [compatibility](https://developer.mozilla.org/docs/Web/SVG/Element/use#Browser_compatibility) for `<use>` is  pretty good. 

Implementing this into my website was pretty straight forward (as shown below), with it currently being used right now as seen on my lovely [Subscribe](/atom.xml) link in my footer.  

**HTML**:
```html
<a href="/atom.xml">
    <svg class="svg-icon icon-orange">
        <use href="/assets/images/icons/simple-icons.svg#rss"></use>
    </svg><span>Subscribe</span>
</a>
```

**CSS:**
```css
.icon-orange {
  color: #f66a0a;
}

.svg-icon {
  box-sizing: initial;
  width: 1.5rem;
  height: 1.5rem;
  display: inline-block;
  fill: currentColor;
  padding: 5px 3px 2px 5px;
  vertical-align: middle;
}
```

**Note:** A little gotcha I encountered (and mostly overlooked when looking at documentation) is you have to reference an \#id somewhere inside your use. So for example:

`<use href="/assets/images/icons/rss-icon.svg"></use>` - **Doesn't work**
<br>*but*<br>
`<use href="/assets/images/icons/rss-icon.svg#rss"></use>` - **Does work** (this requires the `<symbol>` element inside  rss-icon.svg to have an id attached to it so you can reference it in `<use>`)

## CSS `@import` to import style rules from other style sheets

Since I have a separate style sheet for code syntax and another style sheet for printing I thought it would be cool to import them as needed. This was a pretty straight forward thing to add (Only caveat is it needs to be at the top of the style sheet before anything else).

Below is what I am currently using, and if you want a more detailed explanation then I highly suggest you have a look at the Mozilla Developer Docs page on [@import](https://developer.mozilla.org/docs/Web/CSS/@import).

**CSS:**
```css
@import "syntax-highlighting.css";
@import "print.css" print;
```

## Universal box sizing with inheritance

This was shamelessly ~~stolen~~ referenced from the css-tricks [box-sizing](https://css-tricks.com/box-sizing/#article-header-id-3) page.

```css
html { 
    box-sizing: border-box;
}

*,
*:before,
*:after {
    box-sizing: inherit;
}
```

## Global variables in CSS
As always Mozilla Developer docs has come to the rescue again describing how to use [CSS custom properties (variables)](https://developer.mozilla.org/docs/Web/CSS/Using_CSS_custom_properties). This is achieved by defining the custom property on the pseudo-class `:root` (highest-level "parent" element) and then referencing the custom variables via the `var()` functional notation. This is a pretty useful tidbit of information that would probably pay dividends when used in more complex projects.

**CSS:**
```css
:root {
    /* Global Variables */
    --nav-color: #6a9fb5;
}

.site-header, .navigation {
    background-color: var(--nav-color);
}

```

## Customising the width of the tab character!
Not wanting to get into the Spaces VS. Tabs debate I will keep this one quick. [Reference](https://developer.mozilla.org/docs/Web/CSS/tab-size)

```css
pre {
    /* Inital Value was 8, we set it to 4 */
    -moz-tab-size: 4;
    -o-tab-size: 4;
    tab-size: 4;
}
```

## CSS Grid
TODO

## Responsive Video Resize
TODO

## Print Styles
TODO

