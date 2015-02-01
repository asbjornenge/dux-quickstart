# Dux QuickStart

Want to get started with [Dux](https://github.com/asbjornenge/dux)? From the comforts of your own laptop? You can! It's easy!

Follow these simple steps:

## Requirements

* docker
* node.js
* npm

## Liftoff :rocket:

    cdi run 
        --env DUX_DOMAIN=<domain> 
        --dns <dns-host>
        --var DOMAIN=<domain>
        --var DOCKER_HOST=<docker-host> 
        --var DOCKER_PORT=<docker-port> 
        --var FIREBASE_URL=<firebase-url> 
        --var FIREBASE_SECRET=<firebase-secret> 
        containers.json

