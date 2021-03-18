# cpu.fyi

[cpu.fyi](https://cpu.fyi) is a website that lets you link to CPU documentation.

Useful manuals for different CPU architectures are hosted in this repository and presented using [pdf.js](https://mozilla.github.io/pdf.js). The pdf.js frontend provides a few useful features on top of some browsers' default viewer:
- Use back/forward to navigate within a manual.
- Link to specific [pages](https://cpu.fyi/d/4336a7#page=1112), [sections](https://cpu.fyi/d/98dfae#G11.5672154), or [views](https://cpu.fyi/d/32862e#page=200&zoom=auto,-249,316) within manuals with a scheme that works across browsers. Basically anything you can click to navigate, you can also right-click to copy a link.

The goal is to provide links that are appropriate to embed in long-lived source code. As such, links are intended to be immutable and permanent. They will always reference the same document version even when new versions are added.

## Adding a new manual

1. Put the PDF in the right place in `docs/`.

   See "Manuals" below for the directory structure.
2. Run the `makelinks` script in the root to update symbolic links that power the viewer URL.
3. Update `index.html` to link to the new manual as `/d/aabbcc`.

   Only one version of each manual is linked from the root, though this might be reconsidered if there's an important reason why users might need to refer to many versions.

Package up these commits and send them as a PR.

## Repository layout

### Manuals
Manuals are checked into `docs/<vendor>/<document date YYYY-MM-DD or YYYY-MM>/<original filename>.pdf`. The document date should come from the PDF itself.

The intent is to leave documents in the repo forever, although [GitHub pages limits](https://docs.github.com/en/github/working-with-github-pages/about-github-pages#usage-limits) might make that impractical someday. If and when that happens, we may break docs out into another repo but rewire `/d/` links to keep them working.

### `pdf.js`
A copy of `pdf.js` is checked in at `pdfjs/`.

Inside `pdfjs/web` is `viewer2.html`, which is a slightly modified version of `viewer.html`, which sets the default PDF to load based on the page URL:

```diff
--- pdfjs/web/viewer.html
+++ pdfjs/web/viewer2.html
@@ -39,6 +39,17 @@

     <script src="viewer.js"></script>

+    <script>
+      document.addEventListener('webviewerloaded', function() {
+        // Transform `/abc/123` or `/abc/123.html` to `123`
+        const thisPath = new URL(document.URL).pathname;
+        const thisFile = thisPath.slice(thisPath.lastIndexOf('/') + 1).replace(/\.html$/, '');
+        PDFViewerApplicationOptions.set('defaultUrl', '/links/' + thisFile);
+
+        // PDFViewerApplicationOptions.set('workerSrc', '/pdfjs/build/pdf.worker.js');
+     });
+     </script>
+
   </head>

   <body tabindex="1" class="loadingInProgress">
```

### Symlinks

Inside `links/` are symlinks to manuals. Each is named for a prefix of the SHA256 hash of the PDF.

Inside `d/` there is a `<hash prefix>.html` symlink to match each symlink in `links/`. Every one is a symlink to `web/viewer2.html` which, as we noted above, figures out the PDF to render based on the URL where it was loaded.

GitHub Pages will serve `/d/abc123.html` when `/d/abc123` is requested, which we rely on for shorter links.

Since the `pdf.js` viewer expects to find its resources in a peer directory, `/build` is a symlink to `/pdfjs/build`.

Finally, `links/map.txt` is an index that lets users discover the full path for the PDF referenced by a hash prefix.

### Short links

Inside `l/` are some HTML files that act like a very simple short-link service. Names are randomly generated with e.g. `pwgen 5`.
