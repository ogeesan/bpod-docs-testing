# Bpod Documentation
The contents of this repository is used to build the Bpod documentation site at https://sanworks.github.io/docs

> [!WARNING]
> This is a development repository. All contents will eventually exist in a Sanworks repository. The test site is hosted here: https://ogeesan.github.io/bpod-docs-testing/

## Installing and building
[MkDocs](https://www.mkdocs.org/) and the [Material theme](https://squidfunk.github.io/mkdocs-material/) are used to build the site.

`pip install mkdocs-material` will install all of the requirements necessary to build and view the site.

As a quick guide, mkdocs.yml contains the site structure and settings while docs/ contains all of the content used by MkDocs to generate the site.

Each page in the documentation is a [Markdown (.md)](https://www.markdownguide.org/getting-started/) file that you can edit with a simple text editor like Notepad, a Markdown editor, or even in GitHub itself.

In the terminal, `mkdocs serve` creates a local live server where you can preview the site and live changes.

.github/workflows/deploy-site.yml defines a GitHub workflow that will build the site on GitHub pages whenever changes are made to `main`.

<!-- possible use of python autodoc tool in the future -->

## Contributing
All contributions are welcome!

Any problems, from errors, to typos, to unclear phrasing, can be raised as an Issue or Pull Request.

> [!NOTE]
> Discussion and problems with Bpod itself should be directed to the forums at [sanworks.io/forums](https://sanworks.io/forums/). This repository, and its Issues, is for matters relating to the documentation of Bpod on the documentation site.
