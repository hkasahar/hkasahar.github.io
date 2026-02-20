# FORHIRO.md - Building Your Academic Homepage

## Architecture: How It All Fits Together

Think of your website like a well-organized filing cabinet. Jekyll (the static site generator) acts as the librarian who knows exactly where everything goes.

```
hkasahar.github.io/
├── _config.yml          # The master settings file (your librarian's rulebook)
├── _pages/              # Individual pages (about, teaching, CV, publications)
├── _bibliography/
│   └── papers.bib       # Your publications in BibTeX format
├── _data/
│   └── socials.yml      # Social links and contact info
└── assets/
    ├── img/prof_pic.jpg # Your profile photo
    └── pdf/cv.pdf       # Your CV
```

**The magic of al-folio:** You write your publications once in `papers.bib`, and Jekyll Scholar automatically generates the publications page, creates citation links, highlights your name in author lists, and displays journal badges (ECMA, JASA, etc.).

**Navigation flow:**

- `_config.yml` defines site-wide settings (your name, URL, which features are enabled)
- Each page in `_pages/` has a YAML header with `nav: true/false` and `nav_order: N` controlling the menu
- The `selected: true` field in BibTeX entries determines which papers appear on the homepage

## Decisions: Why We Built It This Way

### Why al-folio instead of academicpages?

Your old site used academicpages (a Minimal Mistakes fork). al-folio is purpose-built for academics with:

- **Native BibTeX support** via Jekyll Scholar - no manual HTML for publications
- **Journal abbreviation badges** - ECMA, JASA show as colored badges
- **Award annotations** - your Zellner Award displays automatically
- **Better math support** - MathJax configured out of the box

### Why GitHub Actions deployment instead of branch-based?

GitHub Pages traditionally built from a `gh-pages` branch. The newer GitHub Actions approach:

- Builds with the full Jekyll plugin ecosystem (Jekyll Scholar requires this)
- Gives you build logs to debug failures
- Supports the responsive image processing al-folio uses

### Why we disabled imagemagick

al-folio can automatically generate responsive images (multiple sizes for different screens). We disabled this (`imagemagick: enabled: false`) because:

- It adds build complexity and potential failure points
- Your profile photo is already appropriately sized
- The performance gain is minimal for a text-heavy academic site

## Lessons: The Bugs and How We Squashed Them

### The Great Deployment Mystery

**Problem:** The deploy workflow reported "success" but the site showed old content.

**What happened:** GitHub Pages was configured to deploy from the `master` branch (old content), while we pushed to `main`. Even after switching to "GitHub Actions" deployment, the old cached content persisted.

**The fix:** We updated the deploy workflow to use the official `actions/deploy-pages@v4` instead of `JamesIves/github-pages-deploy-action@v4`. The official action integrates properly with GitHub's new Pages deployment API.

**Lesson:** When GitHub says "deploy from GitHub Actions," it expects the `actions/deploy-pages` action, not third-party alternatives.

### The Environment Protection Wall

**Problem:** Deploy workflow failed with "Branch 'main' is not allowed to deploy to github-pages due to environment protection rules."

**The fix:** Go to repository Settings > Environments > github-pages, and change "Deployment branches" from "Protected branches only" to "All branches" (or add `main` explicitly).

**Lesson:** GitHub's security defaults assume you want branch protection. For a personal site, you can relax these.

### The Missing Einstein

**Problem:** Build kept failing at the Jekyll step with exit code 1.

**Root cause:** The `profiles.md` page referenced `about_einstein.md` (a template example file) which we had deleted.

**The fix:** Simplified `profiles.md` to remove the reference.

**Lesson:** When cleaning up template files, grep for references to deleted files:

```bash
grep -r "deleted_filename" _pages/ _layouts/ _includes/
```

### The Empty Year Field

**Problem:** BibTeX parsing failed silently.

**Root cause:** One working paper had `year = {}` (empty braces).

**The fix:** Added a year: `year = {2005}`.

**Lesson:** BibTeX is unforgiving. Validate your `.bib` file:

```bash
grep -n "= {}" papers.bib  # Find empty fields
```

## Quick Reference: Common Tasks

### Add a new publication

Edit `_bibliography/papers.bib`:

```bibtex
@article{kasahara2025newpaper,
  author = {Kasahara, Hiroyuki and Coauthor, Name},
  title = {Your Paper Title},
  journal = {Journal Name},
  year = {2025},
  doi = {10.xxxx/xxxxx},
  abbr = {ABBR},        # Optional: badge shown on publications page
  selected = {true}     # Optional: show on homepage
}
```

### Update your CV

Replace `assets/pdf/cv.pdf` with your new CV file (keep the same filename).

### Add a course syllabus

1. Upload PDF to `assets/pdf/econ628_syllabus.pdf`
2. Link from `_pages/teaching.md`: `[Syllabus (PDF)](/assets/pdf/econ628_syllabus.pdf)`

### Change profile photo

Replace `assets/img/prof_pic.jpg` (keep the same filename, JPEG format).

### Add Google Scholar link

Edit `_data/socials.yml`:

```yaml
scholar_userid: YOUR_GOOGLE_SCHOLAR_ID
```

## The Build Pipeline

When you push to `main`:

1. GitHub Actions triggers `.github/workflows/deploy.yml`
2. Ruby/Jekyll builds the site, processing BibTeX via Jekyll Scholar
3. The `_site/` folder is uploaded as an artifact
4. `actions/deploy-pages` publishes to GitHub Pages
5. Site is live at https://hkasahar.github.io (usually within 2 minutes)

Check build status: https://github.com/hkasahar/hkasahar.github.io/actions

---

_Last updated: January 2026_
