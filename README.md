# Beargineers FTC Team #27628 Website

A minimal, single-page website for the Beargineers FIRST Tech Challenge robotics team. Built with Jekyll and hosted on GitHub Pages.

## About

This is a simple landing page featuring our team logo and links to our social media presence.

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
- The logo file is `logo.png` in the root directory

## Project Structure

```
beargineers.nl/
├── _config.yml          # Site configuration
├── .github/workflows/   # GitHub Actions for deployment
├── assets/css/          # Custom styling
├── index.md             # Homepage (single page)
├── logo.png             # Team logo
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
- **Update logo**: Replace `logo.png`
- **Modify styling**: Edit `assets/css/style.scss`
- **Update content**: Edit `index.md`
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

Simply replace `logo.png` with your updated logo (PNG format recommended).

### Modify Styling

Edit `assets/css/style.scss` to customize:
- Colors
- Button styles
- Layout and spacing
- Responsive breakpoints

### Add Content

To expand beyond a single page, you can:
1. Create new `.md` files in the root directory
2. Add links in `index.md`
3. Update navigation in `_config.yml` if using a theme with navigation

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
