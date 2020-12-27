# README

[docker-nginx-webdav-nononsense](https://github.com/dgraziotin/docker-nginx-webdav-nononsense) aims to be a Docker image that enables a no-nonsense WebDAV system on the latest available nginx, stable and mainline.

The image, and resulting container, is designed to run behind a reverse proxy (e.g., the great [jc21/nginx-proxy-manager](https://github.com/jc21/nginx-proxy-manager)) to handle SSL. So, it runs on port 80 internally.

## Why no-nonsense?

I'm taking it lightly: my own project is no-nonsense to me ;-) there is nothing wrong with other projects.

Here is what I think sets it apart from other nginx Docker images.

- Based on [linuxserver.io](https://linuxserver.io) Ubuntu. All their magic is here, too, including their handling of user and group permission.
- Takes inspiration (read: copy/pastes) all instructions by [Rob Peck to make WebDAV working well on nginx](https://www.robpeck.com/2020/06/making-webdav-actually-work-on-nginx/), which brings the following goodies:
  1. Includes the latest [nginx-dav-ext-module](https://github.com/arut/nginx-dav-ext-module) (enables PROPFIND, OPTIONS, LOCK, UNLOCK).
  2. Includes the latest [headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module) to handle broken and weird clients.
  3. Includes the latest [ngx-fancyindex](https://github.com/aperezdc/ngx-fancyindex) to make directory listing look good.
- No more [NSPOSIXErrorDomain:100 Error](https://megamorf.gitlab.io/2019/08/27/safari-nsposixerrordomain-100-error-with-nginx-and-apache/) with Safari 14+ on MacOS and on iOS 14+.
- Works (AFAIK) out of the box with [jc21/nginx-proxy-manager](https://github.com/jc21/nginx-proxy-manager), no "Advanced" configuration needed, no `proxy_hide_header Upgrade;` needed.
- CORS headers are all set.
- Some good configuration settings are automatized through env variables (see below).

# Settings

Mount any of these two volumes:

- `./path/to/dir:/data` is the root folder that nginx will serve for WebDAV content (`/data`).
- `./htpasswd:/etc/nginx/htpasswd` is the [Apache HTTP compatible flat file to register usernames and passwords](https://httpd.apache.org/docs/2.4/programs/htpasswd.html). If you provide one, you can tell the container who your username and passwords are. If you provide one, `WEBDAV_USERNAME` and `WEBDAV_PASSWORD` env vars (see below) are ignored. Please note that all users have the same access levels.

These are environment variables you can set, and what they do.

- `PUID=1000` user id with read/write access to `./path/to/dir:/data` volume. Nginx will use the same to be able to read/write to the folder.
- `PGID=1000` group id with read/write access to `./path/to/dir:/data` volume. Nginx will use the same to be able to read/write to the folder.
- `TZ=Europe/Berlin` specifies timezone for the underlying GNU/Linux system.
- `WEBDAV_USERNAME=user` to set a single username to access WebDAV. Ignored if `WEBDAV_PASSWORD`is not set, ignored if `./htpasswd:/etc/nginx/htpasswd` is mounted.
- `WEBDAV_PASSWORD=password` to set the password to the single username to access WebDAV. Ignored if `WEBDAV_USERNAME`is not set, ignored if `./htpasswd:/etc/nginx/htpasswd` is mounted.
- `SERVER_NAMES=localhost,ineed.coffee` comma separated hostnames for the server. 
- `TIMEOUTS_S=1200` expressed as seconds, sets at the same time various nginx timeouts: `send_timeout`, `client_body_timeout`, `keepalive_timeout`, `lingering_timeout`.
- `CLIENT_MAX_BODY_SIZE=120M` limits file upload size to the expressed value, which must end wither with `M`(egabytes) or `G`(igabytes).

# Usage

- Clone this repository, edit the included docker-compose.yml, and run `docker-compose build && docker-compose up` to build and run the container. Access it from http://localhost:32080; or
- Build the Dockerfile and run the container with docker; or
- Pull and run my docker image [dgraziotin/nginx-webdav-nononsense](https://hub.docker.com/repository/docker/dgraziotin/nginx-webdav-nononsense) and use it with docker-compose or docker.

If you are using a reverse proxy (you should!) do not forget to connect the container to the reverse proxy. Follow the instructions of your reverse proxy.
With [jc21/nginx-proxy-manager](https://github.com/jc21/nginx-proxy-manager), I add the following to the docker-compose.yml:

```
networks:
    default:
       external:
         name: reverseproxy
```

Consider also un-exposing the port if you use a reverse proxy.

# Feature requests

I will add features if I happen to need them. To name one, I do not need native SSL support, because I use a reverse proxy.
However, I welcome pull requests.