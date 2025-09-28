---
layout: post
title:  "How I Built This Jekyll Site"
date:   2025-09-28 14:00:00 +0530
categories: jekyll webdev
---

Building this site was a journey from zero to a clean, minimal Jekyll blog. Here's how I did it step by step.

## Starting Point

I wanted a minimal, technical-looking personal site with clean typography, fast loading, and focus on content over design.

## Setting Up Ruby and Jekyll

First challenge: getting Ruby and Jekyll working properly on macOS.

```bash
# Install rbenv for Ruby version management
brew install rbenv ruby-build

# Add to shell config
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(rbenv init -)"' >> ~/.zshrc

# Install Ruby and Jekyll
rbenv install 3.3.0
rbenv global 3.3.0
gem install jekyll bundler
```

The key was using `rbenv` instead of fighting with system Ruby versions. This approach cleanly manages multiple Ruby versions without conflicts.

## Typography: Choosing JetBrains Mono

After researching fonts used by tech sites, I settled on **JetBrains Mono** for everything - headers, body text, and code. It's readable, has that technical aesthetic, and is popular among developers.

The implementation required overriding Jekyll's minima theme:

```scss
// assets/main.scss
---
---
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&display=swap');
$base-font-family: "JetBrains Mono", monospace;
$code-font-family: "JetBrains Mono", monospace;
@import "minima";
@import "custom";
```

## Custom Styling

Created `_sass/custom.scss` for a complete typography hierarchy using JetBrains Mono throughout:

**Typography System:**
- **H1 (post titles)**: 21px, bold, black
- **H2 (section headers)**: 16px, bold, black  
- **H3 (subsections)**: 15px, bold, black
- **Body text**: 14px, regular, gray (#383838)
- **Meta text**: 12px, lighter gray (#666666)
- **Links**: Blue (#0066cc) with hover underlines

**Homepage Specific Adjustments:**
- Post titles: 16px (smaller than article H1s)
- "Posts" heading: 20px with 21px bottom margin
- Post list spacing: 20px between items

The key challenge was overriding Jekyll's minima theme defaults. Required specific selectors and `!important` declarations:

```scss
// More specific than theme defaults
.page-content h1, .post-content h1, h1.post-title {
  font-size: 21px !important;
  font-weight: 700 !important;
  color: #000000 !important;
}

// Homepage-specific styling
.home h2.post-list-heading {
  font-size: 20px !important;
  margin-bottom: 21px !important;
}
```

**Code and Content Styling:**
- Code blocks: Light gray background (#f5f5f5) with 3px border radius
- Blockquotes: Left border accent with indentation
- Lists: Consistent 14px sizing to match body text

## Minimal Footer with Icons

Replaced the default footer with icon-only links:

- Email, GitHub, X/Twitter, RSS
- SVG icons for consistency
- Horizontal alignment with flexbox
- Removed redundant text since RSS is in footer

## Small Details That Matter

**Favicon**: Added a simple sun icon that matches the minimal aesthetic.

**Removed clutter**: 
- Hid "subscribe via RSS" text (redundant with footer icon)
- Reduced post title sizes on homepage
- Cleaned up spacing and alignment

**RSS Feed**: Kept the feed working for tech-savvy readers who use RSS readers.

## The Result

A fast, minimal site that loads quickly and focuses on content. The monospace font gives it a technical feel while remaining highly readable.

**Tech stack:**
- Jekyll static site generator
- GitHub Pages for hosting  
- JetBrains Mono font
- Custom SCSS for styling
- Minimal dependencies

The whole process took an afternoon, but the result is a clean foundation for writing that should last years without needing major updates.

**Live site**: [redoxrux.github.io](https://redoxrux.github.io)