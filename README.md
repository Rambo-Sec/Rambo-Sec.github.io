# RamboSec Jekyll site

This is a GitHub Pages friendly Jekyll rebuild for `rambosec.com`.

## What is included

A cleaner landing page
A certifications page powered by a YAML data file
A blog index page and starter blog posts
An about page
A resume placeholder page that avoids pretending static hosting can securely password protect content
A custom 404 page
A custom domain file for `rambosec.com`

## Local development

Install dependencies:

```bash
bundle install
```

Run locally:

```bash
bundle exec jekyll serve
```

Then open the local server shown by Jekyll, usually `http://127.0.0.1:4000`.

## GitHub Pages deployment

Create a GitHub repository for the site.
Push these files to the default branch.
In repository settings, enable GitHub Pages for the branch.
Set the custom domain to `rambosec.com` in the Pages settings.
Verify the domain in GitHub if you want the extra protection against takeover.
Point DNS at GitHub Pages.

## Files you will probably edit first

`_config.yml`
`_data/certifications.yml`
`_data/social.yml`
`index.md`
`about.md`
`resume/index.md`
Posts in `_posts/`

## Resume page note

The current live site appears to gate the resume page behind a password. A static GitHub Pages site cannot securely protect content server side. For that reason this rebuild uses a placeholder page with safer options, such as:

A direct PDF link stored somewhere you control
A private share link
A contact prompt for recruiter requests

## Suggested next improvements

Replace placeholder verification URLs with your real Credly and Microsoft verification links
Add post images and write up thumbnails
Export your resume to PDF and decide whether you want it public, semi private, or off site
Add more posts from your existing notes
