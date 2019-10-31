# Simple dropbox and file sharing

## Why?

Useful for getting logs and config files to and from servers that you don't have direct SSH access to.

## How

On the server of your choice, install nginx. Then, add a config file into `/etc/nginx/sites-enabled/dropbox`. In this case, this is an AWS instance with the DNS name of files.andyrepton.com. It's using LetsEncrypt for HTTPS.

Add:

```
add_header X-Frame-Options SAMEORIGIN;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";


# HTTPS server
#
server {
    listen *:443 ssl;
    server_name files.andyrepton.com;

    ssl on;
    ssl_certificate         /etc/letsencrypt/live/files.andyrepton.com/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/files.andyrepton.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/files.andyrepton.com/fullchain.pem;

    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 5m;
    ssl_stapling on;
    ssl_stapling_verify on;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";

    ssl_dhparam /etc/nginx/dhparams.pem;
    ssl_prefer_server_ciphers on;

    root /var/www/files;

    location / {
    }

    location /dropbox {
        proxy_pass http://localhost:18000/;
    }
}
```

Then, install python and create the following file:

```
from http.server import HTTPServer, BaseHTTPRequestHandler

from io import BytesIO

class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        body = self.rfile.read(content_length)
        self.send_response(200)
        self.end_headers()
        response = BytesIO()
        self.wfile.write(response.getvalue())
        print(body)


httpd = HTTPServer(('localhost', 18000), SimpleHTTPRequestHandler)
httpd.serve_forever()
```

Send using:

$ cat foo.conf | curl -XPOST -d @- https://files.andyrepton.com/dropbox
