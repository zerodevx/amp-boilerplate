# amp-build

![GitHub release](https://img.shields.io/github/release/zerodevx/amp-build)

An opinionated [AMP](https://amp.dev) static site build and static site generator.

Use this to create full-featured websites that are also AMP-validated pages. Does not require compilation or
a build step during development, because that's so passé. Runs almost entirely off NPM scripts, because things
shouldn't be so complicated.


Features:

* Super optimized build.
* Tailwindcss v1 as utility CSS.
* PurgeCSS v1 to remove unused CSS.
* HTMLMinifier v4 for minification.
* XML Sitemap via static-sitemap-cli v1.
* Material Icons.
* Nested navigation menu.
* Workbox v4 for offline goodness. (WIP)
* App Shell PWA architecture. (WIP)


## Quick-start

### Install

I recommend installing this repo as a `git-subtree`. Here's an [excellent writeup](https://codewinsarguments.co/2016/05/01/git-submodules-vs-git-subtrees/) on this.


```
git subtree add --prefix amp-build https://github.com/zerodevx/amp-build tags/vX.X.X --squash
```

where `X.X.X` is the latest release semver.


Next, install dependencies.

```
cd amp-build
npm install
```

### Initialise

Scaffold the `src` directory.

`npm run scaffold`


### Serve and run

Start the local server.

`npm run serve`

Point your web browser to `http://localhost:8000/src/site/` - no build step required.


### Upgrade

To upgrade `amp-build` to a newer version,

`git subtree pull --prefix amp-build  https://github.com/zerodevx/amp-build tags/vX.X.X --squash`

where `X.X.X` is the newer release semver.



## Usage


### Directory structure

```
src/
  site/           -> site structure goes here
  layouts/        -> page templates go here
  images/         -> images go here
  snippets/       -> reusable HTML chunks go here
  styles/         -> for tailwind to generate styles.css
  scripts/        -> insert-snippet.js
  pwa/            -> PWA-related stuff
```


### Site

During development, by design, everything should work without compilation. Simply run any http server
off the `amp-build` directory at `http://localhost:8000`, or run:

`npm run serve`

Visit your page at `http://localhost:8000/src/site`.

Reference all links by their *absolute* url, for example:

`<a href="http://localhost:8000/src/site/giraffes/">What are giraffes?</a>`


### Snippets

Reusable HTML blocks go into snippets (located at `/snippets`). Insert snippets into page using a `script` tag,
`snippet` attribute, and source to `src/scripts/insert-snippet.js`. For example:

`<script snippet="header.html" src="http://localhost:8000/src/scripts/insert-snippet.js"></script>`


### Default Snippets

```
snippets/header.html          -> Header
snippets/footer.html          -> Footer
snippets/icons.html           -> Icon meta tags
snippets/amp-components.html  -> Default amp-components for every page
snippets/analytics.html       -> Default amp-analytics for every page
```


### Images

All images go into `src/images`. Reference all images by their *absolute* url, for example:

`<amp-img src="http://localhost:8000/src/images/short-giraffe.jpg" width="500" height="500" layout="responsive"></amp-img>`


### Minifying Images

Imagemin takes a long time (and really should only be run once), so minify the images in `src/images/` using:

`npm run build:images`

During build, the `src/images/` directory will be copied directly without further modifications.


### Favicons

I recommend using https://realfavicongenerator.net/ to generate your favicons. Place all favicons into `images/icons`
directory. They will be moved into site root at build. Update `snippets/icons.html` with the associated meta tags.
Note that `images/icons` will be ignored during `imagemin` minification so as to preserve your quality settings.


### Layouts

By default, new pages are generated from `src/layouts/default.html`. This serves as the template whenever a new page
is scaffolded.

Add a new page:

`npm run scaffold:page -- <url> <optional: template>`

For example:

`npm run scaffold:page -- /about/services section`

This copies `src/layouts/section.html` to `src/site/about/services/index.html`. If the directory does not exist,
it will be created.


### Adding Pages Manually

You can add new pages yourself. Simply create the new directory and copy an existing `index.html` in as a reference
template, and ensure that placeholder comments (for eg. `<!--sw:start ... sw:end-->`) remain.

```
mkdir src/site/my-new-page
cp src/site/index.html src/site/my-new-page
```


### Styles

Use tailwind classes to construct your page. If you wish to make changes to `src/styles/tailwind.configjs` (for theming),
or `src/styles/tailwind.css` to add more functional classes - edit the files, then run the following to regenerate
`styles.css`:

`npm run build:tailwind`


### Config

Set global configuration at `src/config.json`.

| Key              | Type          | Description                                                  |
|------------------|---------------|--------------------------------------------------------------|
| stagingUrl       | String        | Base URL of staging site                                     |
| productionUrl    | String        | Base URL of production site                                  |


## Builds

### Test build locally

Generate your build locally into the `build/` directory.

`npm run build`

Build can be tested by pointing your web browser to `http://localhost:8000/build`.

### Staging build

Ensure that `stagingUrl` is defined in `src/config.json`, then run

`npm run build-staging`

This prepares `/build` for Staging - files are minified and amp-validated but have analytics disabled and are not indexable by search engines. The `/build` directory can then be copied over and published to your Staging server.

### Production build

Ensure that `productionUrl` is defined in `src/config.json`, then run

`npm run build-production`

This generates the production build that is minified, amp-validated, analytics enabled, and indexable. The `/build` directory
is ready to be copied over to your Production directory for deployment.

I recommend using [rsync](https://gist.github.com/zerodevx/9f71a4530f70d4947747ad170c67cd25) to perform the copy,
something like:

```
cd build
rsync -rlpgoDci --del --exclude=.DS_Store . ../production
```

### Sitemap for Production

I recommend generating the sitemap after the rsync operation. This preserves the correct `lastmod` timestamp
for files that are not changed, so Google won't reindex those in vain.

```
npm run build:sitemap -- -r <path/to/production>
```

This generates a `sitemap.xml` in your production path.


### amp-cache

The first AMP page that a user enters, on mobile, from a search engine, are viewed through the AMP viewer - which loads
a special cached version of your page from the AMP cache. You might want to invalidate the cache after a new production
deployment.

The amp-cache can be updated via a [update-cache](https://developers.google.com/amp/cache/update-cache) operation.

View instructions from the link to generate your `privateKey.pem` and `apikey.pub` pair. The cache can be updated via

```
npm i --save @ampproject/toolbox-cli
sscli [BASEURL] -r <path/to/root> -t > sitemap.txt
while IFS= read -r line; do amp update-cache $line; done < sitemap.txt
```



## TO-DOs

### LD+JSON

Make a better way to automate this.


### Service Worker

Integrate service worker generation into build.


### PWA/App Shell

Build should eventually become a [PWA](https://developers.google.com/web/progressive-web-apps/), where AMP-validated pages
are indexed by search engines and act as an entry point to a complete PWA app.

At the moment, [Amp Shadow](https://github.com/ampproject/amphtml/blob/master/spec/amp-shadow-doc.md) does not
seem to be working very well, and it's not clear if AMP devs are continuing work on it.

Also, during proof-of-concept it's discovered that not all AMP components work as expected after being piped in.


## Changelog

**v1.1.0** - 2019-08-17:
* Remove `shx` - sorry Windows.
* Update `static-sitemap-cli` to v1.
* Add gist for copy-build best practice.

**v1.0.0** - 2019-07-31:
* Complete overhaul so that things should be easier to reason with now.
* This should be good enough as a base to build on.

**v0.1.0** - 2018-10-05:
* Initial release.
