# MuPDF / WebAssembly

## MuPDF WebAssembly

### Building

The WebAssembly build has only been tested on Linux at the moment.
If you use any other platform, you are on your own.

In order to build this you will need to install the
<a href="https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html">Emscripten SDK</a>
in <tt>/opt/emsdk</tt>.
If you install it elsewhere, you will need to edit the platform/wasm/build.sh
script to point to the appropriate location.

From the MuPDF project, you can run <tt>make wasm</tt> to build the WebAssembly
library. The results of the build are a mupdf-wasm.wasm binary and
mupdf-wasm.js script, placed in platform/wasm/.

In order to build a web application based on MuPDF, you will need to copy
these two files and make them available to your page.

The mupdf-wasm.wasm binary is quite large, because it contains not only the MuPDF
library code, but also the 14 core PDF fonts, various CJK mapping resources,
and ICC profiles. In order to keep it as small as possible, it is built with a
minimal features set that does not include CJK fonts, EPUB support, etc.

### Using

The example script in platform/wasm/mupdf-view.html shows how to use the
MuPDF WebAssembly library.

The first part is including the mupdf-wasm.js script, which pulls in
and instantiates the WebAssembly module:

```html
<script src="mupdf-wasm.js"></script>
```

MuPDF uses the Emscripten virtual file system to load a document, so you will
need to seed it with the file you want to view in a Module.preRun
callback function:

```html
<script>
Module.preRun = function () {
	FS.createPreloadedFile(".", "input.pdf", "input.pdf", true, false);
}
</script>
```

When the filesystem has finished preloading the file data and initialized the
code, it will call the Module.postRun callback. From here, you can
use the 'mupdf' object to call various functions to open the document and
render pages into various formats.

* mupdf.openDocument(filename)

  Open a document and return a handle.

* mupdf.freeDocument(doc)

  Free a document and its associated resources.

* mupdf.documentTitle(doc)

  Return the document title as a string.

* mupdf.documentOutline(doc)

  Return a DOM element containing the table of contents formatted as
  an unordered HTML list with links to pages using anchor fragments "#page%d".

* mupdf.countPages(doc)

  Return the number of pages in the document.

* mupdf.pageWidth(doc, page, dpi) and mupdf.pageHeight(doc, page, dpi)

  Return the dimensions of a page. Page numbering starts at 1.

* mupdf.drawPageAsPNG(doc, page, dpi)

  Render the page and return a PNG image formatted as a data URI,
  suitable for using as an image source attribute.

* mupdf.pageLinks(doc, page, dpi)

  Retrieve an HTML string describing the links on a page,
  suitable for including as the innerHTML of an image map.

* mupdf.drawPageAsSVG(doc, page)

  Return a string with the contents of the page in SVG format.

* mupdf.drawPageAsHTML(doc, page)

  Return a string with the contents of the page in HTML format,
  using absolute positioned elements.

### Example

Here is a very simple example of loading a document and drawing its first page:

```html
<!DOCTYPE html>
<html>
<head>
<script src="mupdf-wasm.js"></script>
<script>
Module.preRun = function () {
	FS.createPreloadedFile(".", "input.pdf", "input.pdf", true, false);
}
Module.postRun = function () {
	var DPI = 96;
	var doc = mupdf.openDocument("input.pdf");
	var img = document.getElementById("page1");
	img.src = mupdf.drawPageAsPNG(doc, 1, DPI);
	var map = document.getElementById("page1map");
	map.innerHTML = mupdf.pageLinks(doc, 1, DPI);
}
</script>
</head>
<body>
<img id="page1" usemap="#page1map">
<map id="page1map" name="page1map"></map>
</body>
</html>
```
