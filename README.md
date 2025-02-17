# Terminado

[![Build Status](https://github.com/jupyter/terminado/actions/workflows/test.yml/badge.svg?query=branch%3Amain++)](https://github.com/jupyter/terminado/actions/workflows/test.yml/badge.svg?query=branch%3Amain++)
[![Documentation Status](https://readthedocs.org/projects/terminado/badge/?version=latest)](http://terminado.readthedocs.io/en/latest/?badge=latest)

This is a [Tornado](http://tornadoweb.org/) websocket backend for the
[Xterm.js](https://xtermjs.org/) Javascript terminal emulator library.

It evolved out of [pyxterm](https://github.com/mitotic/pyxterm), which
was part of [GraphTerm](https://github.com/mitotic/graphterm) (as
lineterm.py), v0.57.0 (2014-07-18), and ultimately derived from the
public-domain [Ajaxterm](http://antony.lesuisse.org/software/ajaxterm/)
code, v0.11 (2008-11-13) (also on Github as part of
[QWeb](https://github.com/antonylesuisse/qweb)).

Modules:

- `terminado.management`: controls launching virtual terminals,
  connecting them to Tornado's event loop, and closing them down.
- `terminado.websocket`: Provides a websocket handler for
  communicating with a terminal.
- `terminado.uimodule`: Provides a `Terminal` Tornado [UI
  Module](http://www.tornadoweb.org/en/stable/guide/templates.html#ui-modules).

JS:

- `terminado/_static/terminado.js`: A lightweight wrapper to set up a
  term.js terminal with a websocket.

Local Installation:

> $ pip install -e .\[test\]

Usage example:

```python
import os.path
import tornado.web
import tornado.ioloop
# This demo requires tornado_xstatic and XStatic-term.js
import tornado_xstatic

import terminado
STATIC_DIR = os.path.join(os.path.dirname(terminado.__file__), "_static")

class TerminalPageHandler(tornado.web.RequestHandler):
    def get(self):
        return self.render("termpage.html", static=self.static_url,
                           xstatic=self.application.settings['xstatic_url'],
                           ws_url_path="/websocket")

if __name__ == '__main__':
    term_manager = terminado.SingleTermManager(shell_command=['bash'])
    handlers = [
                (r"/websocket", terminado.TermSocket,
                     {'term_manager': term_manager}),
                (r"/", TerminalPageHandler),
                (r"/xstatic/(.*)", tornado_xstatic.XStaticFileHandler,
                     {'allowed_modules': ['termjs']})
               ]
    app = tornado.web.Application(handlers, static_path=STATIC_DIR,
                      xstatic_url = tornado_xstatic.url_maker('/xstatic/'))
    # Serve at http://localhost:8765/ N.B. Leaving out 'localhost' here will
    # work, but it will listen on the public network interface as well.
    # Given what terminado does, that would be rather a security hole.
    app.listen(8765, 'localhost')
    try:
        tornado.ioloop.IOLoop.instance().start()
    finally:
        term_manager.shutdown()
```

See the [demos
directory](https://github.com/takluyver/terminado/tree/master/demos) for
more examples. This is a simplified version of the `single.py` demo.

Run the unit tests with:

> $ pytest
