# Woojar - Tech & Engineering Blog

A telecom software engineer's tech blog built with Hugo using a custom "TelecomTech" theme.

## Prerequisites

- [Hugo](https://gohugo.io/getting-started/installing/) (extended version for SCSS support)
- Git

## Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/woojar/woojar-blog.git
   cd woojar-blog
   ```

2. Start the development server:
   ```bash
   hugo server -D
   ```

3. Visit `http://localhost:1313` to view the site

## Building the Site

To generate the static site files:

```bash
hugo
```

This will create the static files in the `public/` directory.

## Deployment Options

### GitHub Pages (Recommended)

1. Create a new repository on GitHub (e.g., `username.github.io`)

2. Add the GitHub Pages deployment script to your repository:
   ```bash
   #!/bin/bash
   # deploy.sh
   
   # Build the site
   hugo
   
   # Go to the public folder
   cd public
   
   # Add changes to git
   git add .
   
   # Commit changes
   msg="Rebuilding site $(date)"
   if [ -n "$*" ]; then
       msg="$*"
   fi
   git commit -m "$msg"
   
   # Push to GitHub Pages
   git push origin main
   ```

3. Set up GitHub Actions workflow (`.github/workflows/gh-pages.yml`):
   ```yaml
   name: GitHub Pages
   
   on:
     push:
       branches: [ main ]
   
   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
       - uses: actions/checkout@v2
         with:
           submodules: true
           fetch-depth: 0
       
       - name: Setup Hugo
         uses: peaceiris/actions-hugo@v2
         with:
           hugo-version: 'latest'
           extended: true
       
       - name: Build
         run: hugo
       
       - name: Deploy
         uses: peaceiris/actions-gh-pages@v3
         with:
           github_token: ${{ secrets.GITHUB_TOKEN }}
           publish_dir: ./public
   ```

### Netlify

1. Sign up at [Netlify](https://netlify.com)
2. Connect your GitHub repository
3. Set the build command to `hugo` and publish directory to `public`
4. Deploy!

### Vercel

1. Sign up at [Vercel](https://vercel.com)
2. Import your Git repository
3. Set the build command to `hugo` and output directory to `public`
4. Deploy!

## Custom Theme

This site uses a custom "TelecomTech" theme designed specifically for telecom engineers:
- Modern, responsive design with dark mode support
- Professional blue/teal color scheme
- Optimized for technical content and code snippets
- Mobile-first approach

## Content Management

- Posts are located in `content/posts/`
- Add new posts with `hugo new posts/post-title.md`
- Tags and categories are automatically handled

## Updating the Site

1. Make changes to content or theme
2. Test locally with `hugo server -D`
3. Build with `hugo`
4. Commit and push changes to deploy

## Project Structure

```
.
├── archetypes/        # Content templates
├── content/           # Blog posts and pages
├── themes/            # Custom theme files
├── public/            # Generated site (not tracked)
├── .gitignore         # Git ignore rules
└── hugo.toml          # Hugo configuration
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request