# My Tech Blog

A personal tech blog built with Hugo and the PaperMod theme.

## Quick Start

### Writing a New Post
To create a new blog post, run:
```bash
hugo new posts/my-new-post.md
```

### Local Development
To preview the site locally (including drafts), run:
```bash
hugo server -D
```
The site will be available at `http://localhost:1313`.

### Deployment Pipeline
This blog is configured with a GitHub Actions workflow (`.github/workflows/deploy.yml`). 
Every time you push changes to the `main` branch, the workflow will automatically build the Hugo site and deploy it to GitHub Pages.

#### Initial GitHub Pages Setup:
1. Go to your GitHub repository in the browser.
2. Navigate to **Settings** > **Pages**.
3. Under **Build and deployment** > **Source**, select **GitHub Actions**.
4. Push your code to the `main` branch. The action will run and your site will be live!
