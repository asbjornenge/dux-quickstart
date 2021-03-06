# Dux QuickStart

> WARNING: The Dux ref. implementation is still quite fragile and constantly changing.

Want to get started with [Dux](https://github.com/asbjornenge/dux)? From the comforts of your own laptop? You can! It's easy!

## Requirements

* [docker](https://www.docker.com/)
* [node/io.js](https://iojs.org/en/index.html)
* [npm](https://www.npmjs.com/)

## Getting ready

### Tools

First, let's install some required tools. We are going to use the [cdi](https://www.npmjs.com/package/cccf-docker-instructions) cli to run our docker containers.

    npm install -g cccf-docker-instructions

### Configure Docker

Dux is an architecture intended for multi-host (even multi-cloud) deployments. In this type of scenario we need to talk to the docker daemon over the wire. You probably can get this to work with a unix socket, mounting volumes and things; I'll leave it to you to figure that out if you really really want to :stuck_out_tongue_closed_eyes:. Let's [bind the docker daemon to a tcp host:port](https://docs.docker.com/articles/basics/#bind-docker-to-another-hostport-or-a-unix-socket). 

If you are running docker as a service; exactly how to modify the docker configuration differ between distributions so I won't go into details about it. Google is your friend.

### Firebase :fire:

The [dux-statestore](https://github.com/asbjornenge/dux-statestore) stores it's state in a [firebase](https://www.firebase.com/). This is mostly for convenience but it also enables some cool things like multi-cloud and GUIs. They are free for development puposes, so go ahead and make one. You will need the **url** and the **secret**.

### Service discovery

We need some way for our Dux services (and application services) to discover each other. This implementation of Dux uses DNS based service discovery, just like the internet. It has the advantage that applications don't need to change to leverage it. We are going to use [rainbow-dock](https://github.com/asbjornenge/rainbow-dock) :rainbow: 

### Containers

All the containers required to run Dux are described in the [containers.json](https://raw.githubusercontent.com/asbjornenge/dux-quickstart/master/containers.json) file in this repository. Go ahead and download it.

    wget https://raw.githubusercontent.com/asbjornenge/dux-quickstart/master/containers.json

Alright, I think thats about it! You ready?

![popcorn](http://i.giphy.com/UlW9P3FfFEkXS.gif)

## Liftoff :rocket:

Let's run those containers!!

    cdi run 
        --dns <docker-host>                        // Adds the --dns flag to all containers.     (172.17.42.1)
        --var DOMAIN=<domain>                      // Pick a domain, any domain ~ popcorn.wtf
        --var DOCKER_HOST=<docker-host>            // The host where you bound the docker daemon (172.17.42.1)
        --var DOCKER_PORT=<docker-port>            // The port you bound the docker daemon to    (4243)
        --var FIREBASE_URL=<firebase-url>          // The url to the firebase you created
        --var FIREBASE_SECRET=<firebase-secret>    // The secret to the firebase you created
        containers.json

**PS!** If you have any other containers running, now would be a good time to stop them. Dux has a scheduler component that will kill and remove any containers it does not know about.

If all goes well, you should have 5 containers running.

    $ docker ps
    CONTAINER ID        IMAGE                                       ..   NAMES
    172bf1db3feb        asbjornenge/dux-statestore:latest           ..   statestore               
    3324085b06ff        asbjornenge/dux-scheduler:latest            ..   scheduler                
    f08f039e09d3        asbjornenge/dux-dispatcher:latest           ..   dispatcher               
    c2a039b111f8        asbjornenge/rainbow-dock-populator:latest   ..   rainbow-dock-populator   
    6ae0d008188a        asbjornenge/rainbow-dock:3.0.1              ..   rainbow-dock  

If not; [log an issue](https://github.com/asbjornenge/dux-quickstart/issues).

## Manipulate State

Now that we have our Dux containers up and running, let's add some state!

First, let's verify that the **StateStore** is ready.

![statestore_log1](statestore_log1.png)

As you can see, it initially has some trouble resolving other services ip addresses, but after a short while the DNS service is up and populated, and they are ready.  
Let' see how the **Scheduler** is doing.

![scheduler_log1](scheduler_log1.png)

The scheduler is reporting ready too, but as we can see, it's looking for the state **/containers** and **/hosts** that it currently can't find.

Let's add it!

You can add nested objects to firebase by clicking the little green +. The *0* indidates that this is the first element in an array.

![firebase_add_hosts](firebase_add_hosts.png)

Notice the statestore picked up the new state and distributed it.

![statestore_distribute_hosts](statestore_distribute_hosts.png)

The scheduler is still quiet, it needs some containers too!

![firebase_add_containers](firebase_add_containers.png)

Now things should start happening on the scheduler! Don't worry if you see some blank output while pulling the image. It's a bug in the docker cli that should be fixed i 1.4. Once the image is pulled the scheduler will start 5 elasticsearch containers!

![scheduler_run_containers](scheduler_run_containers.png)

5 might be a bit too much, so let's scale that down. Just modify the state in firebase and watch the :sparkles: unfold.

![scheduler_kill_containers](scheduler_kill_containers.png)

Yellow for kill, red for remove. :tada:

## End of the road 

And that's about as far as this quickstart will take you.

I hope you play around, experiment and contribute! The implementation is still very crude & fragile. Both the **Scheduler** and the **StateStore** needs work. Maybe even a re-implementation. And there are lots of other dux-tools to make. Your imagination is the limit!

enjoy
