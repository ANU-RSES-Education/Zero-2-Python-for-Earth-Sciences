# EMSC - introduction to Programming

## Web pages

- [Web version (Basic Examples Only)](https://anu-rses-education.github.io/Zero-2-Python-for-Earth-Sciences/book)


## Binder:

 - [Australian Research Data Commons Binder](https://binderhub.rc.nectar.org.au/v2/gh/ANU-RSES-Education/Zero-2-Python-for-Earth-Sciences/main?labpath=WebBook/BinderLandingPage.qmd)
 - [www.mybinder.org](https://mybinder.org/v2/gh/ANU-RSES-Education/Zero-2-Python-for-Earth-Sciences/HEAD?labpath=WebBook/BinderLandingPage.qmd)

## Local Development

### Building the book

The book uses Quarto with interactive Python examples powered by pyodide (Python in the browser via WebAssembly).

To build the book:

```bash
quarto render
```

The built book will be in `_build/book/`.

### Previewing the book locally

Because the book uses pyodide for interactive Python examples, you need to serve it over HTTP (not just open the HTML files directly). Use the included server script:

```bash
python serve-book.py
```

This will:
- Start a local HTTP server (default port 8000)
- Automatically open your browser to view the book
- Auto-select an available port if 8000 is in use

Press `Ctrl+C` to stop the server.

### Requirements

- Quarto
- Python 3.x (for the preview server)
