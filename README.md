# Ginger & Co. Events

Official website for Ginger & Co. Afrobeats Fitness and Lifestyle events in Vienna.

## Features

- ✅ Responsive design for all devices
- ✅ Events page with detailed information
- ✅ Video testimonials with modal popup
- ✅ Event video showcase
- ✅ Modern Afrobeats-inspired design
- ✅ Optimized images via Cloudinary CDN

## Deployment

### Option 1: GitHub Pages (Recommended)

1. Go to your repository on GitHub
2. Click **Settings** → **Pages**
3. Under "Build and deployment":
   - **Source**: Deploy from a branch
   - **Branch**: Select `claude/netlify-deployment-setup-011CUcnrMAPPuKT9iv67VPr5`
   - **Folder**: `/ (root)`
4. Click **Save**
5. Your site will be live at: `https://spendbox.github.io/Gingerco/`

### Option 2: Netlify

1. Go to [netlify.com](https://netlify.com) and sign in
2. Click **Add new site** → **Import an existing project**
3. Connect to GitHub and select the `Gingerco` repository
4. Configure build settings:
   - **Branch**: `claude/netlify-deployment-setup-011CUcnrMAPPuKT9iv67VPr5`
   - **Build command**: (leave empty)
   - **Publish directory**: `.` (auto-detected from netlify.toml)
5. Click **Deploy site**

The netlify.toml configuration is already set up for optimal deployment.

## Site Structure

- `index.html` - Home page with about, events preview, and community testimonials
- `events.html` - Detailed events page for Vienna Takeover 2025
- `netlify.toml` - Netlify deployment configuration
- `.nojekyll` - Ensures GitHub Pages doesn't process files with Jekyll
- `.gitignore` - Git ignore rules

## Latest Updates

- Modal video player for testimonials
- Updated Cloudinary media URLs
- New hero banner for events page
- Vienna Takeover 2025 video recap

---

© 2025 Ginger & Co. All rights reserved.
