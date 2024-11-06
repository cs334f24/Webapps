# Moravian Web Apps

This repo details how to deploy and make applications deployable on the moravian webapps system.

## Setup/Installation

1. Install [Nginx](https://github.com/nginx/nginx?tab=readme-ov-file#downloading-and-installing) on the host machine according to your distributions instructions. For `apt` based systems:

```bash
apt install nginx-full python3-certbot-nginx
```

2. Clone this repository and copy configuration files over to `/etc/nginx/`

```bash
git clone https://github.com/cs334f24/Webapps
cd Webapps
cp webapps /etc/nginx/sites-available
cp -r webapps-mixins /etc/nginx/
cp -r webapps-available /etc/nginx/
mkdir /etc/nginx/webapps-enabled
```

3. Disable the default site

```bash
rm /etc/nginx/sites-enabled/default
```

4. Update the server configuration in `/etc/nginx/sites-available/webapps` by setting `<subdomains.domain.tld>` to the url to serve content out of (ex. `learn-git.cs.moravian.edu`) and `<your-dashboard-root>` to the directory to serve an application dashboard out of (ex. `/home/webapps/dashboard`

Before proceeding any further with the Nginx config, you must first setup the OAuth2 service OAuth2 Proxy as described below.

5. Generate secret keys for OAuth2 Proxy

```bash
cd oauth2-proxy
python3 -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())' > oauth_client_secret.env
python3 -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())' > secure_cookie_secret.env
```

6. Install and enable the OAuth2 Proxy Socket

```bash
cp oauth2-proxy.socket /etc/systemd/system
systemctl enable --now oauth2-proxy.socket
```

7. Launch OAuth2 Proxy

### Recommendations

It is recommended that [Docker](https://www.docker.com/) is installed to help manage web applications.

## Administrators

## Developers

* to be deployable on `webapps.cs.moravian.edu` your application must be able to accept traffic via a TCP socket (standard ip address and port combination) or a Unix Domain Socket (a path to a socket file on the system)
* TCP sockets are easier to configure, but can have accidental collusions due to ports
* Unix Domain sockets require additional application configuration, but provide better performance per request

* you provide two files, a configuration file for the location of the server and one for the configuration of routes


### Common Use Cases

#### Absolute Paths

If you are using absolute paths in your application you must prepend them with `/<your_app>`.
For flask the most convenient way is to set the environment variable `SCRIPT_NAME` to `/<your_app>`, then adjust any hardcoded absolute urls to use `url_for` in a template.
Here is an example

<details>
<summary>templated html example</summary>

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='style.css') }}">
    <!-- DO NOT DO THIS <link rel="stylesheet" type="text/css" href="/static/style.css"> -->
    <!-- Remote resources do not need to be adjusted -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
...
</body>
```


</details>

Alternatively for purely static content the html `<base>` element can be used.
[See MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base) for more details.

#### OAuth

