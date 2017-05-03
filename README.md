<!-- Copyright 2017 ForgeRock AS. -->
# Quick Reference Card for ForgeRock DS/OpenDJ Servers and Tools

This refcard was created using a build of ForgeRock Directory Services 4.1.
YMMV. See the product documentation for full details:
<https://backstage.forgerock.com/docs/ds>.

## Building Output

View the Markdown version at <https://github.com/markcraig/opendj-refcard/blob/master/refcard.md>.

LaTeX files need to be converted to an output format, such as PDF.
The `.tex` reference cards build with `pdflatex`.
For example, on macOS:

```
# Install pdflatex, etc.
brew cask install basictex
open /usr/local/Caskroom/basictex/2016.1009/mactex-basictex-20161009.pkg
pdflatex refcard*.tex
open refcard*.pdf
```

## Implementation Notes

The LaTeX format files differ slightly in order to fit the server-side use cases on one double-sided page.

The Markdown version currently has example lines that do not wrap,
but it is so much easier to maintain that the LaTeX format might not be worth the effort.

* `refcard-client.tex`: LaTeX format, client-side use, one double-sided PDF.
* `refcard-server.tex`: LaTeX format, server-side use, one double-sided PDF.
* `refcard.md`: Markdown format,
* `refcard.tex`: LaTeX format, client and server-side use, multiple page PDF.
* `tools.properties`: Example default settings file.
