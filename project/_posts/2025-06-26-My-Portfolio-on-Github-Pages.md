---
layout:     post
title:      "Launching My Portfolio Website on GitHub Pages"
sitemap: false
excerpt_separator: <!--more-->
hide_last_modified: true
comments: true
---

# Launching My Portfolio Website on GitHub Pages

## Introduction

Building a personal portfolio has been on my wishlist for a while. This month I finally shipped the **first version** of my site—hosted on **GitHub Pages** and powered by the **Jekyll**. It’s still a work‑in‑progress, but it already feels good to have a single place where I can curate everything from low‑level firmware experiments to full‑stack AI applications.

<!--more-->

## Motivation & Background

My journey into web tooling started last year when I helped the **Arm Education Team** stand up the *Developer Labs* site. Although I’m not a dedicated front‑end engineer, that experience demystified static‑site generators and gave me a crash course in production HTML/CSS. Since then, internships and hackathons have nudged me further toward the full‑stack world, and spinning up a portfolio felt like the natural next step.

## Tech Stack at a Glance

| Layer                     | Choice           | Why It Works                                                            |
| ------------------------- | ---------------- | ----------------------------------------------------------------------- |
| **Static‑Site Generator** | **Jekyll**       | Mature, Ruby‑based, seamless GitHub Pages support                       |
| **Theme**                 | **HydeJack**     | Polished, responsive, minimal tweaks needed to look good out‑of‑the‑box |
| **Hosting**               | **GitHub Pages** | Free HTTPS, CI/CD via `gh‑pages`, version‑controlled                    |
| **Comments**              | **Disqus**       | Low‑friction feedback channel, optional social login                    |

## Features in Version 1.0

1. **Project Showcase** – Dedicated pages for everything from bare‑metal C projects to Next.js apps. Each entry has a hero image, tech stack badges, and a short write‑up.
2. **Responsive Layout** – HydeJack’s grid adapts from smartphones to 4K monitors with almost no manual CSS.
3. **Disqus‑Powered Discussion** – Drop a comment or idea right under each post—no account required.
4. **Continuous Deployment** – Every `main` push triggers GitHub Actions to rebuild and publish.

## Implementation Highlights

### 1. Fork → Rewrite → Polish

I began by forking HydeJack’s demo, stripping the demo content. Tweaking `_config.yml` handled 90% of the branding work—site title, description, social links, favicon, etc.

### 2. Custom Styling with *Just Enough* CSS

Most custom styling lives in `_sass/my-style.scss`. My goal was to stay opinionated but lightweight:

```scss
.notice {
  background-color: #dbeff1;        
  border-left: 4px solid #4FB1BA;   
  padding: 16px;                    
  margin: 20px 0;                  
  border-radius: 4px;             
  line-height: 1.5;              
  position: relative;           
  text-align: center;
}

.thread:empty + .notice {
  display: block;
}

.thread + .notice {
  display: none;
}
```

### 3. Disqus Integration

```html
<!-- _layouts/post.html (excerpt) -->

{% assign disqus = site.disqus | default:site.disqus_shortname %}
{% if disqus %}
<div class="thread" id="disqus_thread"></div>
<div class="notice">
    <a>Please refresh the page to view the comments.</a>
</div>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
<script>!function(w, d) {
  if (d.getElementById("disqus_thread")) {
    if (w.DISQUS) {
      w.DISQUS.reset({
        reload: true,
        config() {
          this.page.url = w.location.href;
          this.page.title = d.title;
        },
      });
    } else {
      w.disqus_config = function disqusConfig() {
        this.page.url = w.location.href;
        this.page.title = d.title;
      };
      w.loadJSDeferred(d.getElementById("_hrefDisqus").href + '/embed.js');
    }
  }
}(window, document);</script>
{% endif %}
```

## Challenges & What I Learned

* **Theme vs. Originality** – HydeJack looks great, but avoiding the *"default demo site"* vibe required deliberate content and visual tweaks.
* **SEO Basics** – YAML front‑matter matters. Meta descriptions, canonical URLs, and Schema.org tags all help search engines take me seriously.
* **CSS Debugging** – Chrome DevTools is now my best friend. Fixing a stubborn mobile navbar taught me more flexbox than any tutorial ever did.

## Roadmap

* **Analytics** – Opt‑in privacy‑minded telemetry (probably Plausible).
* **More Posts** – Low‑level software deep‑dives, hackathon retrospectives, and AI side quests are in the pipeline.

## Conclusion

This site is the living documentation of my developer journey. I’ll keep refining the design, adding posts, and experimenting with new tech. If something sparks an idea, drop a comment—I’d love to hear your feedback!

> *Thanks for reading, and stay tuned for the evolution!*
