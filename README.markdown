# The JSON-Driven Schema

These are the slides for the talk "The JSON-Driven Schema".

It uses [reveal.js](http://lab.hakim.se/reveal-js/) to render (mostly) markdown
files into a slideshow. The slides contain code, and that code is intended to
be executed as part of the talk.

To run this slideshow locally, you need to serve the contents of this repo from
some kind of httpd. The simplest way I've found to do that is to run this
command from the root of this repo:

```bash
$ python -m SimpleHTTPServer
```

This will start a server at [http://localhost:8000](http://localhost:8000).

## Table of Contents

The slides are split into separate markdown files, which are intended to be
viewed in this order:

1. [Introduction](introduction.markdown)
2. [JSON and PostgreSQL](json-postgres.markdown)
3. [GitHub API overview](github-api.markdown)
4. [Loading JSON into PostgreSQL](loading.markdown)
5. [Looking at JSON objects](objects.markdown)
6. [Flattening JSON objects](flatten.markdown)
7. [Converting scalar data types](scalars.markdown)
8. [Normalizing](extract-table.markdown)
9. [Extracting a link table](link-table.markdown)

## License

![cc-by-sa](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)
The presentation content is available under the [Creative Commons
Attribution-ShareAlike 4.0 International
License](http://creativecommons.org/licenses/by-sa/4.0/).

[reveal.js](https://github.com/hakimel/reveal.js) is Copyright (C) 2016 Hakim
El Hattab, http://hakim.se.
