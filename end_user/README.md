# End-user documentation

Just open `./docs/end_user/docs/index.md`. This will be available online once
repository is public. To view it locally, you first need to build it and open
file in your web browser.

If building from source to update documentation (see
[here](https://www.mkdocs.org/) how to write documentation):

1. Make sure you have dependancies: `pip install mkdocs mkdocs-material`

2. Run in `./docs/end_user` directory `mkdocs build` command. Open then
   `./docs/end_user/index.html`. Alernativelly run `mkdocs serve`. This serves
   documentation through server that reloads on file changes (useful when
   editing documentation).
