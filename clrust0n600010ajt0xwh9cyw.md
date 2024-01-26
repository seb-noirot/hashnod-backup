---
title: "Understanding CSS Specificity: A Concise Guide"
seoTitle: "CSS Specificity Guide: Fix Style Conflicts"
seoDescription: "Explore CSS specificity with this clear guide. Learn to solve style conflicts effectively with practical examples and tips"
datePublished: Fri Jan 26 2024 15:28:57 GMT+0000 (Coordinated Universal Time)
cuid: clrust0n600010ajt0xwh9cyw
slug: understanding-css-specificity-a-concise-guide
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/6JVlSdgMacE/upload/947e17277b428df2101175c4dfdf4128.jpeg
tags: css, specificity, basic-of-html-and-css

---

Today at work, we encountered a common but often overlooked challenge in web development: CSS specificity. We had a global `font-size` definition that, to our surprise, wasn't being overridden by a more recent class style. Upon inspecting the CSS in the developer tools, it became clear that the issue was rooted in CSS specificity - the new class style had a lower specificity score than the global style.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1706282610982/4e7424dc-785e-40f6-9356-8f0b919d74d2.png align="center")

This real-world problem underscored the importance of understanding CSS specificity, not just in theory but in practical, day-to-day web development. In this article, I aim to demystify CSS specificity, drawing on this experience to show how a deeper understanding of this concept can save time and frustration in your web development journey.

**The Basics of CSS Specificity**

Specificity in CSS is like a game where different selectors earn different points. The selector with the highest points wins and its styles get applied. Here’s a quick rundown of the points system:

* Element selectors (e.g., `div`, `h1`): 1 point
    
* Class selectors (e.g., `.container`, `.header`): 10 points
    
* ID selectors (e.g., `#navbar`, `#footer`): 100 points
    
* Inline styles (e.g., `style="font-size:12px"`): 1000 points
    

**Example:**

```css
div { /* 1 point */ }
.header { /* 10 points */ }
#navbar { /* 100 points */ }
```

**Calculating Specificity**

When selectors are combined, their points add up. The one with the highest total wins.

**Example:**

```css
#navbar .menu-item { /* 100 (ID) + 10 (Class) = 110 points */ }
body #content .article h2 { /* 1 (Element) + 100 (ID) + 10 (Class) + 1 (Element) = 112 points */ }
```

In the examples above, the second selector has higher specificity, so its styles will override those of the first selector if they target the same element.

**Practical Tips and Best Practices**

* **Avoid Overusing ID Selectors**: IDs are great for JavaScript hooks or unique styling, but they can make your CSS hard to override due to their high specificity.
    
* **Leverage Classes**: Classes strike a good balance between specificity and reusability. They are your best friend for most styling tasks.
    
* **Use** `!important` Sparingly: This declaration overrides any other declarations, regardless of their specificity. Overuse can lead to maintenance nightmares. Use it only as a last resort.
    

CSS specificity might seem complex at first, but once you understand the basics, it becomes a powerful tool in your web development arsenal. Experiment with different selectors, and you'll soon get a feel for how specificity shapes your stylesheets.

**Further Reading/Resources**

* [MDN Web Docs on Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
    
* [CSS Tricks: Specifics on CSS Specificity](https://css-tricks.com/specifics-on-css-specificity/)