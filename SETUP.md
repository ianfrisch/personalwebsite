# Ian Frisch Website - Setup Guide

This is a Jekyll site with Decap CMS for easy content management, designed for GitHub Pages.

## Quick Start

### 1. Create GitHub Repository

1. Create a new repository on GitHub named `ianfrisch.github.io` (or `username.github.io`)
2. Push this code to the repository:

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/USERNAME/USERNAME.github.io.git
git push -u origin main
```

### 2. Enable GitHub Pages

1. Go to repository **Settings** → **Pages**
2. Under "Build and deployment", select:
   - Source: **GitHub Actions**
3. Create `.github/workflows/jekyll.yml` (see below)

### 3. Add Book Cover Images

Add cover images to `assets/images/`:
- `inside-the-cartel-cover.jpg`
- `magic-is-dead-cover.jpg`

You can download these from the original Squarespace site or use new images.

### 4. Set Up Decap CMS Authentication

This is the trickiest part. You have two options:

#### Option A: Use Netlify Identity (Easiest)

1. Deploy to Netlify instead of GitHub Pages (free tier works)
2. Enable Netlify Identity in site settings
3. Update `admin/config.yml`:

```yaml
backend:
  name: git-gateway
  branch: main
```

#### Option B: GitHub OAuth with External Proxy

1. Create a GitHub OAuth App:
   - Go to GitHub → Settings → Developer settings → OAuth Apps → New OAuth App
   - Application name: "Ian Frisch CMS"
   - Homepage URL: `https://ianfrisch.github.io`
   - Authorization callback URL: `https://api.netlify.com/auth/done`

2. Use Netlify's free OAuth proxy:
   - Create a free Netlify account
   - Create a new site (can be empty)
   - Go to Site settings → Access control → OAuth
   - Install GitHub provider with your OAuth app credentials

3. Update `admin/config.yml`:

```yaml
backend:
  name: github
  repo: USERNAME/USERNAME.github.io
  branch: main
  base_url: https://api.netlify.com
  auth_endpoint: auth
```

#### Option C: Self-hosted OAuth Proxy

Deploy your own OAuth proxy using [netlify-cms-github-oauth-provider](https://github.com/vencax/netlify-cms-github-oauth-provider) on a service like Heroku, Railway, or Render.

### 5. Update Configuration

Edit `admin/config.yml` and replace:
- `OWNER/REPO` with your GitHub username/repo
- `OWNER.github.io` with your actual site URL

Edit `_config.yml` if you need to change:
- Site title, description, email
- Social media usernames
- Base URL (if not using root domain)

---

## GitHub Actions Workflow

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
          ruby-version: '3.2'
          bundler-cache: true
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Build with Jekyll
        run: bundle exec jekyll build
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

---

## Local Development

1. Install Ruby and Bundler
2. Run:

```bash
bundle install
bundle exec jekyll serve
```

3. Visit `http://localhost:4000`

For CMS local testing, run:

```bash
npx decap-server
```

Then uncomment `local_backend: true` in `admin/config.yml`.

---

## Using the CMS

Once authentication is set up:

1. Go to `https://yoursite.com/admin/`
2. Log in with GitHub
3. You'll see:
   - **Books**: Add/edit book entries
   - **Pages**: Edit About, Copywriting, Film/TV, Journalism pages
   - **Settings**: Update site title, description, social links

---

## File Structure

```
├── _books/              # Book entries (editable via CMS)
├── _layouts/            # Page templates
├── admin/               # Decap CMS files
│   ├── config.yml       # CMS configuration
│   └── index.html       # CMS interface
├── assets/
│   ├── css/style.css    # Site styles
│   └── images/          # Images (upload via CMS)
├── _config.yml          # Jekyll configuration
├── about.md             # About page
├── copywriting.md       # Copywriting page
├── film-tv.md           # Film/TV page
├── journalism.md        # Journalism page
└── index.html           # Homepage (Books)
```

---

## Customization

### Changing Colors

Edit CSS variables in `assets/css/style.css`:

```css
:root {
  --color-primary: #1a1a1a;
  --color-accent: #c9a227;
  /* etc. */
}
```

### Changing Fonts

Update the Google Fonts link in `_layouts/default.html` and the font variables in CSS.

---

## Troubleshooting

**CMS shows "Error loading the CMS configuration"**
- Check that `admin/config.yml` is valid YAML
- Ensure repo path is correct

**CMS login fails**
- Verify OAuth app credentials
- Check callback URL matches exactly

**Site not building**
- Check GitHub Actions tab for errors
- Ensure `Gemfile.lock` is committed

---

## Finding Themes

If your friend wants a different look, here are resources for Jekyll themes:

- **Jekyll Themes**: https://jekyllthemes.io/
- **GitHub Topics**: https://github.com/topics/jekyll-theme
- **Jamstack Themes**: https://jamstackthemes.dev/ssg/jekyll/

Popular portfolio themes:
- [Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes)
- [Hyde](https://github.com/poole/hyde)
- [Forty](https://github.com/andrewbanchich/forty-jekyll-theme)
- [Freelancer](https://github.com/jeromelachaud/freelancer-theme)

To use a different theme, replace the `_layouts/` folder and `assets/css/` with the theme files, then update `_config.yml` as needed.
