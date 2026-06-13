# Beargineers FTC Team #27628 Website

A Jekyll website for the Beargineers FIRST Tech Challenge robotics team. It combines team identity, learning resources, open engineering projects, CAD downloads, blog posts, match records, and downloadable engineering portfolios.

## About

This site is intended to be useful beyond a season portfolio. The current structure is:

- `learn/`: curated FTC learning resources
- `projects/`: open-source software and engineering projects
- `cad/`: reusable CAD drops and design notes
- `blog/`: team stories, build logs, and competition notes
- `portfolio/`: downloadable engineering portfolio PDFs and supporting pages
- `matches/`: one-page season and event-organized match archive
- `team/`: team information

## Local Development

### Prerequisites

- Ruby (version 2.7 or higher, works with Ruby 4.0+)
- RubyGems
- Bundler

### Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/beargineers/beargineers.nl.git
   cd beargineers.nl
   ```

2. **Install dependencies:**
   ```bash
   bundle install
   ```

3. **Run the development server:**
   ```bash
   bundle exec jekyll serve
   ```

4. **View the site:**
   Open your browser and navigate to `http://localhost:4000`

### Development Tips

- Changes to most files will automatically rebuild the site
- Changes to `_config.yml` require restarting the server
- The logo file is `assets/images/logo.svg`

## Project Structure

```
beargineers.nl/
├── _config.yml          # Site configuration
├── .github/workflows/   # GitHub Actions for deployment
├── _cad/                # CAD drops and design notes
├── _layouts/            # Jekyll layouts
├── _matches/            # Event match archives, organized by season folder
├── _portfolios/         # Engineering portfolio entries
├── _posts/              # Blog posts
├── _projects/           # Open-source project pages
├── _resources/          # Learning resource pages
├── assets/css/          # Custom styling
├── assets/images/       # Pictures
├── assets/              # Other resources like PDFs
├── portfolio-archive/   # Legacy portfolio-derived support pages by season/game
├── index.md             # Homepage
├── *.md                 # Top-level landing and archive pages
├── Gemfile              # Ruby dependencies
├── CNAME                # Custom domain configuration
└── README.md            # This file
```

## Making Changes (Team Members)

### 1. Clone the Repository

```bash
git clone https://github.com/beargineers/beargineers.nl.git
cd beargineers.nl
```

### 2. Set Up Your Environment

```bash
bundle install
```

### 3. Make Your Changes

Test your changes locally:
```bash
bundle exec jekyll serve
```

Then view at `http://localhost:4000`

Common tasks:
- **Update logo**: Replace `assets/images/logo.svg`
- **Modify styling**: Edit `assets/css/style.scss`
- **Update homepage**: Edit `index.md`
- **Add learning material**: Add a Markdown file in `_resources/`
- **Add project page**: Add a Markdown file in `_projects/`
- **Add CAD drop**: Add a Markdown file in `_cad/`
- **Add blog post**: Add a dated Markdown file in `_posts/`
- **Add match event**: Add a Markdown file in `_matches/<season>/`
- **Update social links**: Edit `_config.yml`

### 4. Push Your Changes

```bash
git add .
git commit -m "Description of your changes"
git push origin main
```

The site will automatically rebuild and deploy to [beargineers.nl](https://beargineers.nl) in 1-2 minutes.

## Deployment

The site is automatically deployed via GitHub Actions:
- Every push to the `main` branch triggers a rebuild
- The site is deployed to [beargineers.nl](https://beargineers.nl)
- Deployment typically takes 1-2 minutes

## Customization

### Update Site Information

Edit `_config.yml`:
```yaml
team_number: "27628"
instagram_username: beargineers_
github_username: beargineers
```

### Replace Logo

Replace `assets/images/logo.svg` with an updated SVG logo. Use WebP for large photographic assets.

### Modify Styling

Edit `assets/css/style.scss` to customize:
- Colors
- Button styles
- Layout and spacing
- Responsive breakpoints

### Add Content

To add a new top-level page, create a `.md` file in the root directory and include `nav_title` plus `nav_order` in the front matter when it should appear in the main navigation.

## Ruby 4.0+ Compatibility

This project includes compatibility gems for Ruby 4.0+:
- `csv` - CSV file handling
- `base64` - Base64 encoding
- `logger` - Logging functionality
- `bigdecimal` - Arbitrary precision decimal arithmetic
- `kramdown-parser-gfm` - GitHub Flavored Markdown parser

## Links

- **Live Site**: [beargineers.nl](https://beargineers.nl)
- **Instagram**: [@beargineers_](https://www.instagram.com/beargineers_)
- **GitHub**: [github.com/beargineers](https://github.com/beargineers)

## Support

- Jekyll documentation: [jekyllrb.com/docs](https://jekyllrb.com/docs/)
- GitHub Pages documentation: [docs.github.com/en/pages](https://docs.github.com/en/pages)

## License

This website is for the Beargineers FTC Team #27628. Content and design © Beargineers Team.
