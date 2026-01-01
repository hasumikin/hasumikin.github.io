# hasumikin.com

## Local Development

1. Clone this repository
2. Install dependencies:
   ```bash
   npm install
   ```
3. Add your articles to the `dist/articles/` directory
4. Build the site:
   ```bash
   rake build
   ```
5. Start the local server:
   ```bash
   rake server
   ```
6. Open http://localhost:8000 in your browser

For CSS development, you can watch for changes:
```bash
npm run watch:css
```

### Writing Articles

Create markdown files like `articles/yyyy-mm-dd-slug.md` with YAML front matter:

```markdown
---
title: My First Post
date: 2025-12-24
---

This is the content of my first post.
```

## Adding Misc Pages

You can add custom pages (like About, Contact, etc.) without modifying Buddie:

1. Create a markdown file in the `misc/` directory:
   ```bash
   # Create misc/about.md
   ```

2. Add content with front matter:
   ```markdown
   ---
   title: About This Blog
   ---

   # About This Blog

   Your content here...
   ```

3. Build the site:
   ```bash
   rake build
   ```

4. The page will be accessible at `/about` (or whatever you named the file)

**Example:**
- `dist/misc/about/index.html` → `example.com/about`
- `dist/misc/contact/index.html` → `example.com/contact`
- `dist/misc/privacy/index.html` → `example.com/privacy`

The build process automatically generates a dynamic HTML page for each `.md` file in the `dist/misc/` directory.

### Deployment to GitHub Pages

This project is configured to automatically deploy to GitHub Pages using GitHub Actions.

1. Push your repository to GitHub
2. Enable GitHub Pages in your repository settings:
   - Go to Settings > Pages
   - Source: GitHub Actions
3. Push to the main/master branch to trigger deployment

The workflow will:
- Generate article index
- Download PicoRuby WASM from npm
- Build and deploy to GitHub Pages

## URL Structure

- `/` - Home page (article list, page 1)
- `/page2` - Article list, page 2
- `/yyyy/mm/dd/my_first_post.html` - Individual article from `articles/yyyy-mm-dd-slug.md`
- `/about` - About page (from `misc/about.md`)

## Build Tasks

```bash
# Build CSS with Tailwind
npm run build:css

# Watch CSS for changes during development
npm run watch:css

# Generate article index
rake generate_index

# Generate pages from dist/misc/*.md
rake generate_pages

# Build CSS with Tailwind (via rake)
rake build_css

# Build site for deployment (runs all generation tasks including CSS)
rake build

# Start local development server
rake server
```

## Requirements

- Ruby 3.4 or later
- Node.js 18.x or later
- Rake
- JSON gem

For local development:
```bash
gem install rake
npm install
```

For GitHub Actions deployment, dependencies are installed automatically.

## License

MIT

Copyright (c) 2025 HASUMI Hitoshi (@hasumikin)
