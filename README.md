# Clowdy

Clowdy is a simplified way to develop applications with docker.

The only thing you need to use Clowdy is a config file named `clowdy.config.js` in the root directory of your project.


## Service Mode (prod, dev, test)

One of the core concepts in Clowdy is the mode that a service is operating in. By default all services are run in `prod` mode.


## Config file (clowdy.config.js)

The config file is just a plain javascript file that describes the various services in your project.

**clowdy.config.js**

    const path = require('path');
    
    // Set the project name. You don't need to do this. The project name will
    // be set to the current directory name by default.
    project('awesome-app');
    
    // define a service named redis
    // services that do not specify a mode are assumed to be "prod"
    service('redis', {
      // use the latest redis docker image from docker hub
      image: 'redis',
      // tell clowdy that this service will listen for connections on port 27017
      expose: [27017]
    });
    
    // build an image named node-prod from the directory at ./node-prod
    // this will expect a file name Dockerfile to exist in ./node-prod
    image('node-prod', path.resolve(__dirname, 'node-prod'));
    
    service('api', 'dev', {
      image: 'node:10-alpine',
      command: ['yarn', 'dev'],
      cwd: '/src',
      environment: {
        NODE_ENV: 'development'
      },
      expose: [3000],
      links: {
        redis: 'redis'
      },
      volumes: {
        '/src': __dirname
      }
    });
    
    service('api', 'prod', {
      image: 'node-prod',
      environment: {
        NODE_ENV: 'production'
      },
      expose: [3000],
      links: {
        redis: 'redis'
      }
    });

**Commands**

Run `clowdy dev api -Ea`  to start up the api service in dev mode, expose all ports, and attach to the container. To detach without sending keystrokes to the process, use `control-d`.

Run `clowdy start api -E` to shut down the dev mode and start the prod mode container. This will automatically build the `node-prod` image from the directory specified in the configuration of the image.

Run `clowdy destroy` to clean up everything.


## Better experience for node.js development


    /*
      What this does:
      - run a nodejs:10-alpine container
      - mount the curent directory at /usr/src/app
      - sets NODE_ENV=development
      - runs "yarn install"
      - runs "yarn dev"
      - watches package.json and runs yarn install on changes
      - watches code and restarts the process on changes
    */
    plugin('@clowdy/nodejs').dev('api', {
      // add any extra configuration
      expose: [3000]
    });
    
    // to mount the ./packages/api directory instead of the current directory
    plugin('@clowdy/nodejs').dev('api', './packages/api', {
      environment: {
        TZ: 'America/New_York'
      },
      expose: [8080]
    });
    
    /*
      What this does:
      - same as dev (above)
      - sets NODE_ENV=test
      - runs "yarn install"
      - runs "yarn test"
    */
    plugin('@clowdy/nodejs').test('api', {
      environment: {
        // if you use something that has it's own watcher (mocha, jest, etc.)
        // then you can disable the file watcher
        WATCH_FILES: 'false'
      }
    });
    
    /*
      What this does:
      - creates a new docker image based on node:10-alpine
        - this image runs "yarn install" and "yarn build"
        - it then runs "yarn install --production" and copies that code
          to a fresh image (using multi-stage builds)
        - sets the command to "yarn start"
      - sets NODE_ENV=production
    */
    plugin('@clowdy/nodejs').prod('api', {
      expose: [80]
    });


## Run a Minecraft Server with Clowdy

Create a `clowdy.config.js` file somewhere. I put it in my home directory (to hold all of my personal services).

We’ll use the `itzg/minecraft-server` image published to docker hub. The file should look something like this:

    const path = require('path');
    
    project('global');
    
    service('minecraft', {
      environment: {
        EULA: 'TRUE'
      },
      expose: [25565],
      image: 'itzg/minecraft-server',
      volumes: {
        '/data': volume('minecraft')
      }
    });

Now run `clowdy start minecraft -E` from the command line in the directory containing the clowdy config file.

That’s it! =-)

**Other things to do**


- Checking status: `clowdy status`
- See the logs: `clowdy logs minecraft`
- To poke around the container in a shell: `clowdy shell minecraft`
- Stopping the server: `clowdy stop minecraft`
- Destroying the container and the minecraft server data: `clowdy destroy`

