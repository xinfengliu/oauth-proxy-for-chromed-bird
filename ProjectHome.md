This is a very uninteresting project. Its only purpose is to have freedom to use twitter.

## Configuration for [chromed bird](http://chromedbird.com/): ##

```
Twitter API url: http://YourApp.appspot.com/api/
Oauth url: http://YourApp.appspot.com/api/oauth/

API signing url: https://api.twitter.com/
Oauth signing url: https://api.twitter.com/oauth/
```

## Source code (It's so small, not worth wasting a sourcecode repository). ##

```
::::::::::::::
Entry.py
::::::::::::::
from google.appengine.ext import webapp
from google.appengine.ext.webapp.util import run_wsgi_app

from Proxy import Proxy
from Oauth import Oauth

application = webapp.WSGIApplication( [(r'/api/oauth/.*', Oauth),
                                       (r'/api/.*', Proxy)
                                      ]
                                    )

def main():
    run_wsgi_app(application)

if __name__ == "__main__":
  main()
::::::::::::::
Oauth.py
::::::::::::::
from Proxy import Proxy
import re

class Oauth(Proxy):
    def handle_content(self, source):
        return re.sub("https://api.twitter.com/", "http://YourApp.appspot.com/api/", source)



::::::::::::::
Proxy.py
::::::::::::::
from google.appengine.ext import webapp
from google.appengine.api import urlfetch

class Proxy(webapp.RequestHandler):
    def doProxy(self):
            url = "https://api.twitter.com" + self.request.path_qs[4:]
            self.request.headers.update({'content-type':'application/x-www-form-urlencoded'})
            data = urlfetch.fetch(url, payload=self.request.body, method=self.request.method, headers=self.request.headers)
            self.response.status = data.status_code
            self.response.headers = data.headers
            self.response.out.write(self.handle_content(data.content))

    def post(self):
        self.doProxy()

    def get(self):
        self.doProxy()

    def put(self):
        self.doProxy()

    def delete(self):
        self.doProxy()

    def handle_content(self, source):
        return source

::::::::::::::
app.yaml
::::::::::::::
application: YourApp
version: 1
runtime: python
api_version: 1

handlers:
- url: /api/.*
  script: Entry.py
- url: /.*
  static_files: index.html
  upload: index.html
::::::::::::::
index.html
::::::::::::::
Good Luck.
::::::::::::::
```