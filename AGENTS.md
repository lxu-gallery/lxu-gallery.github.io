# AGENTS.md

Working agreements for AI coding agents in this repository.

## What this project is

Leon Xu's personal photo gallery, live at <https://lxu-gallery.github.io>. A
static Jekyll site that GitHub Pages builds and serves straight from `master` —
there is no CI and no build step of our own. **A push to `master` is a
deploy.**

It is a template, not an application: the "Multiverse" design by
[HTML5 UP](https://html5up.net), via a Jekyll port by João Neto. Keep changes
small and in the template's own style. A framework, a bundler, or a rewrite
does not belong here.

| Path | Holds |
|---|---|
| `_images/NNN.md` | One file per photo: front matter `title` and `caption`, no body |
| `assets/images/fulls/NNN.jpeg` | Full-size photo, opened in the popup |
| `assets/images/thumbs/NNN.jpeg` | Thumbnail shown in the grid |
| `_layouts/multiverse.html` | The whole gallery page; loops over `site.images` in filename order |
| `_includes/` | `head`, `header`, `footer`, `social`, `favicons` |
| `assets/js/main.js` | Template script: turns thumbnails into CSS backgrounds, wires the popup and the About panel |
| `assets/js/jquery.min.js`, `jquery.poptrox.min.js` | Vendored libraries |
| `assets/css/main.css` | **Compiled** from `assets/sass/`; Jekyll does not compile the Sass |
| `assets/favicons/` | Icons and `site.webmanifest` |
| `404.html` | Standalone page with its own inline styles |

The layout builds both image URLs from the `.md` filename and hardcodes the
`.jpeg` extension, so a photo's three files must share its number, and the
images must be `.jpeg`, not `.jpg`.

Do not hand-edit `assets/css/main.css`; it would drift from the Sass it was
compiled from. A style that belongs to one page goes in that page, as in
`404.html`.

## Adding a photo

1. Take the next free number, zero-padded to three digits.
2. Add `assets/images/fulls/NNN.jpeg` and `assets/images/thumbs/NNN.jpeg`,
   compressed first (tools are listed in `README.md`).
3. Add `_images/NNN.md`:

   ```yaml
   ---
   title: Mt. Hood Afternoon
   caption: Portland, OR, USA
   ---
   ```

4. Check that neither image carries GPS coordinates. The repository and the
   site are public, and a location tag on a photo near home publishes it. No
   photo in the repository has one today; keep it that way.

   ```bash
   python3 -c 'import sys; from PIL import Image
   for f in sys.argv[1:]: print(f, "HAS GPS" if Image.open(f).getexif().get_ifd(0x8825) else "clean")' \
     assets/images/fulls/NNN.jpeg assets/images/thumbs/NNN.jpeg
   ```

## Known issues — leave them unless asked

These look broken and are, but the owner has chosen to leave them for now. Do
not fix them on the side of another change:

- The "Get in touch" form in `_includes/footer.html` posts to the site itself;
  GitHub Pages answers 405 and the message is lost.
- Photo 025 has both images but no `_images/025.md`, so it is not shown.
- Captions mix `US` and `USA`, and some titles end in a trailing space.

## Environment

Nothing is installed in the repository and nothing needs to be. To preview a
change, install the gem set GitHub Pages uses **outside** the repository and
build from there:

```bash
mkdir -p /tmp/gallery-jekyll
cat > /tmp/gallery-jekyll/Gemfile <<'EOF'
source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins
gem "ffi", "< 1.17"   # newer ffi needs Ruby >= 3.0; the macOS system Ruby is 2.6
EOF
export BUNDLE_GEMFILE=/tmp/gallery-jekyll/Gemfile BUNDLE_PATH=/tmp/gallery-jekyll/vendor
export PAGES_REPO_NWO=lxu-gallery/lxu-gallery.github.io
bundle install
bundle exec jekyll serve --destination /tmp/gallery-site --baseurl "" --port 4000 --no-watch
```

Each odd piece is there for a reason:

- **`PAGES_REPO_NWO`**: the `origin` remote is an SSH host alias, not
  `github.com`, so `jekyll-github-metadata` cannot work out the repository and
  the build fails without it.
- **`--baseurl ""`**: without GitHub API access the same plugin guesses a base
  path of `/pages/lxu-gallery`. The real site is served from `/`.
- **`--destination`** outside the repository, so build output never lands in
  the working tree.
- The committed `Gemfile` does not install on the system Ruby, because current
  `ffi` needs Ruby 3. Use the recipe rather than editing it.

This resolves to Jekyll 3.9.5; GitHub Pages runs 3.10.0. That is close enough
for templates. Say so if a change depends on the difference.

**Do not install or upgrade system packages to build this site**: no
`brew install`, no `sudo gem install`. On this machine `brew install ruby`
started compiling LLVM and Rust from source, and the shared library it upgraded
on the way broke `node`. If the recipe stops working, stop and ask.

## Hygiene

- **Everything here is public**, both the repository and the site. No secrets,
  tokens, or absolute paths (`/Users/...`) in committed files. The email
  address in `_config.yml` is public on purpose.
- **Photos are © Leon Xu, all rights reserved.** Keep the photo notice at the
  top of `LICENSE`. Keep the "Design: HTML5 UP" credit in the footer: the
  template's CC BY 3.0 license requires it.
- **Do not re-encode, resize, rename, or delete photos without asking.**
  Image quality is the owner's call, and a removed photo leaves the live site at
  the next push.
- **Vendored JavaScript** is updated by replacing the file. Download it from two
  sources (for jQuery, `code.jquery.com` and jsDelivr), check that the hashes
  match each other and the project's published integrity hash, then test the
  popup. `jquery.poptrox` is old, and jQuery 4 removes APIs it may rely on, so
  stay on jQuery 3.
- `AGENTS.md` is listed under `exclude` in `_config.yml` so the site does not
  publish it. A new top-level file that is not for visitors goes there too.

## Git

**Do not create commits unless explicitly asked.** This includes `git commit`,
`git commit --amend`, and anything else that writes history. Do not push:
pushing `master` publishes the site. Do not create branches or open PRs on
your own initiative.

Read-only git is always fine: `git status`, `git diff`, `git log`, `git show`,
`git blame`.

Leave finished work in the working tree and say what changed. The user commits.

## Before claiming work is done

1. Build with the recipe above and read the output: no Liquid errors, no
   warnings you did not expect.
2. For any change to a layout, include, stylesheet, or script, load the page in
   a browser (headless Chromium through Playwright works) and check:
   - every thumbnail renders: each `.thumb > a.image` gets a `background-image`;
   - clicking a thumbnail opens the popup with the full image and caption;
   - the header link opens the About panel;
   - there are no console errors and no 404s for assets.
3. `git status` shows only the files you meant to change.

Do not infer success from having written the code. If something fails, say so
with the output. If part of the scope was skipped, say that explicitly.
Checks that need a real device or service, such as the link preview in WeChat
or iMessage or pinch-zoom on a phone, are manual and done by the owner.
