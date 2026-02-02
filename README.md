# Beargineers FTC Team #27628 Website

Official Jekyll-based website for the Beargineers FIRST Tech Challenge robotics team.

## About

This website showcases our team's journey, robot development, competitions, and outreach activities. Built with Jekyll and designed to be hosted on GitHub Pages.

## Local Development

### Prerequisites

- Ruby (version 2.7 or higher)
- RubyGems
- GCC and Make

### Setup Instructions

1. **Install dependencies:**
   ```bash
   bundle install
   ```

2. **Run the development server:**
   ```bash
   bundle exec jekyll serve
   ```

3. **View the site:**
   Open your browser and navigate to `http://localhost:4000`

### Development Tips

- Changes to most files will automatically rebuild the site
- Changes to `_config.yml` require restarting the server
- Posts should be placed in the `_posts` directory with the naming format: `YYYY-MM-DD-title.md`
- Images should be stored in an `assets/images/` directory

## Project Structure

```
beargineers.nl/
├── _config.yml          # Site configuration
├── _posts/              # Blog posts
├── _team_members/       # Team member profiles (optional)
├── _layouts/            # Custom layouts (if needed)
├── _includes/           # Reusable components (if needed)
├── assets/              # CSS, images, JavaScript
├── index.md             # Homepage
├── about.md             # About page
├── team.md              # Team page
├── robot.md             # Robot information
├── contact.md           # Contact page
├── blog.md              # Blog listing page
├── Gemfile              # Ruby dependencies
└── README.md            # This file
```

## Deployment to GitHub Pages

### Option 1: GitHub Pages with main branch

1. **Create a GitHub repository** named `beargineers.nl` or `username.github.io`

2. **Push your code:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit: Beargineers website"
   git branch -M main
   git remote add origin https://github.com/yourusername/beargineers.nl.git
   git push -u origin main
   ```

3. **Enable GitHub Pages:**
   - Go to repository Settings → Pages
   - Set Source to "Deploy from a branch"
   - Select branch: `main` and folder: `/ (root)`
   - Click Save

4. **Custom domain (optional):**
   - Add a `CNAME` file with your domain name
   - Configure DNS settings with your domain provider

### Option 2: GitHub Actions (recommended for more control)

Create `.github/workflows/jekyll.yml`:

```yaml
name: Deploy Jekyll site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
          bundler-cache: true
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Build with Jekyll
        run: bundle exec jekyll build --baseurl "${{ steps.pages.outputs.base_path }}"
        env:
          JEKYLL_ENV: production
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Customization

### Update Site Information

Edit `_config.yml` to update:
- Team name and number
- Contact information
- Social media links
- Site description

### Add Team Members

Create files in `_team_members/` directory:

```yaml
---
name: John Doe
role: Team Captain
bio: Passionate about robotics and leading our team to success.
---
```

### Create Blog Posts

Add new posts to `_posts/` directory with this format:

```markdown
---
layout: post
title: "Your Post Title"
date: 2025-01-15 10:00:00 +0000
categories: [competition, robot-design]
author: Team Member Name
---

Your post content here...
```

### Styling

- The site uses the Minima theme by default
- To customize styles, create `assets/css/style.scss`
- To override theme templates, copy them from the theme to your project

## Content To-Do List

- [ ] Update team member information in `team.md`
- [ ] Add actual team member profiles to `_team_members/`
- [ ] Update contact information and social media links
- [ ] Add robot specifications and photos to `robot.md`
- [ ] Create additional blog posts about your season
- [ ] Add team photos to `assets/images/`
- [ ] Update meeting location and schedule in `contact.md`
- [ ] Create sponsor information and sponsorship packet
- [ ] Add competition schedule and results

## Support

For Jekyll help: https://jekyllrb.com/docs/
For GitHub Pages help: https://docs.github.com/en/pages

## License

This website is for the Beargineers FTC Team #27628. Content and design © Beargineers Team.
