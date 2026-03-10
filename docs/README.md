# GitHub Pages Deployment Configuration

This folder contains the configuration for deploying the XR Educational Bookmark Platform documentation to GitHub Pages.

## Files

- **index.html** - Main landing page for the website
- **_config.yml** - Jekyll configuration (optional, as we use custom HTML)
- **.nojekyll** - Tells GitHub Pages to not process with Jekyll
- **[Documentation files]** - Copied from root directory

## Automatic Deployment

The GitHub Actions workflow (`.github/workflows/deploy-docs.yml`) automatically:

1. Watches for pushes to `main` or `master` branches
2. Detects changes to markdown files or documentation
3. Copies all documentation to this folder
4. Deploys to GitHub Pages

## Manual Repository Configuration

To enable GitHub Pages on your repository:

### Step 1: Repository Settings
1. Go to your GitHub repository
2. Click **Settings** → **Pages**
3. Under "Build and deployment"
   - **Source:** Select "GitHub Actions"
   - (The workflow will handle deployment automatically)

### Step 2: Verify Custom Domain (Optional)
If using a custom domain:
1. Add your domain in the Pages settings
2. Create a CNAME file with your domain

### Step 3: Check Deployment
1. Go to **Actions** tab
2. You should see "Deploy Documentation to GitHub Pages" workflow
3. Wait for it to complete (green checkmark)
4. Access at: https://nunososorio.github.io/XR_EDU

## Manual Deployment (if workflow fails)

If you need to manually deploy:

```bash
# Copy documentation files
cp ../README.md ./index.md
cp ../*.md ./

# Commit and push
git add .
git commit -m "Update documentation"
git push origin main
```

## Troubleshooting

### Pages not updating after push
1. Check the Actions tab for workflow status
2. Ensure `.github/workflows/deploy-docs.yml` exists
3. Verify GitHub Pages source is set to "GitHub Actions"
4. Clear browser cache and wait 5-10 minutes

### Custom domain not working
1. Ensure CNAME file exists in `docs/` folder
2. Verify DNS records are pointing to GitHub Pages
3. Check repository settings for the domain
4. Wait up to 24 hours for DNS propagation

### Markdown not rendering
1. The landing page is custom HTML, not Jekyll
2. Documentation markdown files will be served as-is
3. For better markdown rendering, consider using a Jekyll theme or markdown renderer

## Customization

To customize the landing page:
1. Edit `docs/index.html`
2. Modify colors, content, or links
3. Push to trigger automatic deployment

---

**Deployment Status:** Configured for GitHub Pages  
**Website:** https://nunososorio.github.io/XR_EDU  
**Last Updated:** March 10, 2026
