# blog.wernecki.xyz

Personal technical blog powered by Jekyll.

## Content Focus

- AI-assisted development and MCP integration
- Homelab infrastructure and self-hosting
- Test automation with Playwright
- DevOps and infrastructure as code

## Local Development

### Prerequisites

- Ruby 2.7+ and Bundler
- Jekyll

### Setup

```bash
# Install dependencies
bundle install

# Run local server
bundle exec jekyll serve

# Or with live reload
bundle exec jekyll serve --livereload
```

Visit: `http://localhost:4000`

### Create New Post

```bash
# Create file in _posts/ with format: YYYY-MM-DD-title.md
touch _posts/$(date +%Y-%m-%d)-my-new-post.md
```

Post template:
```markdown
---
layout: post
title: "Your Post Title"
date: 2025-01-12 10:00:00 +0100
categories: [category1, category2]
tags: [tag1, tag2, tag3]
---

Your content here...
```

### Drafts

Posts in `_drafts/` won't be published until moved to `_posts/` with a date.

View drafts locally:
```bash
bundle exec jekyll serve --drafts
```

## Deployment

### Option 1: GitHub Pages (Automatic)

1. Push to GitHub repository
2. Enable GitHub Pages in repository settings
3. Set source to `main` branch
4. Configure custom domain: `blog.wernecki.xyz`

### Option 2: Manual Build

```bash
# Build site
bundle exec jekyll build

# Output is in _site/
# Upload _site/ contents to your web server
```

## DNS Configuration

To use `blog.wernecki.xyz`:

**For GitHub Pages:**
```
Type: CNAME
Name: blog
Value: ch0nkster.github.io
```

**For custom server:**
```
Type: A
Name: blog
Value: [Your server IP]
```

## Site Structure

```
.
├── _config.yml          # Site configuration
├── _posts/              # Published posts
├── _drafts/             # Draft posts
├── assets/              # CSS, images, etc.
├── about.md             # About page
├── index.md             # Home page
└── Gemfile              # Ruby dependencies
```

## Writing Tips

- Use descriptive titles
- Add relevant tags and categories
- Include code examples with syntax highlighting
- Link to your GitHub projects
- Add images to `assets/images/`
- Keep posts focused and scannable

## Maintenance

### Update Dependencies

```bash
bundle update
```

### Check for Issues

```bash
bundle exec jekyll doctor
```

### Build Production Site

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

## Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [Minima Theme](https://github.com/jekyll/minima)
- [Markdown Guide](https://www.markdownguide.org/)

---

**Author**: Marcin Wernecki
**GitHub**: [@ch0nkster](https://github.com/ch0nkster)
**LinkedIn**: [marcinwernecki](https://linkedin.com/in/marcinwernecki)
