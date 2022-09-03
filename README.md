# CLMP-localhost

A dockerised https localhost dev environment stack for PHP/MySQL, ideal for SPA development.
It is additionally configured to allow connecting to and between a locally running service on port 5173 (which could e.g. be a VueJS front-end SPA).
The hard work was done in this ticket: https://caddy.community/t/how-to-get-dockerised-caddy-to-use-self-signed-certs-for-local-dev-with-php-fpm-spa-vuejs/16802/19

## Overview

The architecture here is as follows. Docker creates a network between three containers: Caddy, MySQL and PHP. Caddy is a webserver. The job of caddy is to map the PHP back-end to the Javascript front-end.

## Raison d'être

Localhost dev environments have become a bit more complicated due to recent changes in browser CORS policies, the result of which is that session cookies will not be stored by the browser on localhost unless the TCP protocol in use is https.

### Why Caddy?

- Caddy is written in Go!, which is multi-threading and lightning fast.
- It handles https automagically, including generating all the necesary certificatees for localhost
- You can do everything Apache or Nginx can but the configuration is much simpler

### Caddyfile (config)

Look at the Caddyfile to see the webserver config. It's just a few lines of JSON. There are three main directives:

- debug: just provides more verbose output to stdout on the docker process
- fe.mnr.localhost: defines the reverse proxy to whatever is listening on http port 5173 (e.g. your VueJS app running as a dev server)
- be.mnr.localhost: defines the PHP back-end listener

Note that the two domains (front-end and back-end) must be fully qualified; i.e. they must contain at least two dots. You can make them whatever you want as long as they contain two dots (i.e. min. 3-parts to the domain name) and provided that the last part is `localhost`. For this example we've used `fe.dev.localhost` for the front-end and `be.dev.localhost` for the back-end.

## Installation

In order to see this in action, you need the following:

1. Create an (e.g.) index.php file (or checkout a working PHP api) into the folder defined in the Caddyfile `root` directive for the back-end listener.
1. Add the FQDN's to your /etc/hosts file, pointed at 127.0.0.1, then flush your DNS cache.
1. Launch the containers with the `docker-compose up` CLI command, run from the root of this project (i.e. the folder this README file is in).
1. Add the generated root cert file to your browser's trusted store cache. This can be found in `caddy/data/caddy/pki file
1. Run `curl` or browse to your back-end domain to see that in action.
1. Start your front-end dev server listening on port 5173. In this case `cd api/fe; npm install; npm run dev`
1. Browse to your front-end FQDN (NB: not http://localhost:5173) to see the front-end in action.

## Usage

In order to build your API, just add your PHP application in the `app/api` folder and add your front-end code into the `app/fe` folder, from where you'll run the dev server using whatever technique usual
