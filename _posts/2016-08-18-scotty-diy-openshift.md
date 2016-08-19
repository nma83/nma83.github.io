---
layout: post
title: Running a Scotty web-app in Openshift
---

This post talks about running a web-app written in Haskell using the
[Scotty](https://github.com/scotty-web/scotty) framework, on
[Openshift](openshift.redhat.com) servers.

# Create the Openshift app

Create a new app in the Openshift web console using the
[DIY](https://developers.openshift.com/languages/diy.html) cartridge. We use the
DIY cartridge for 2 reasons:

* No ready-made Haskell cartridge in Openshift
* Building a Haskell app from source on a [small gear](https://developers.openshift.com/managing-your-applications/resource-management.html)
 would certainly run into capacity issues and crash

Select a public URL for the app. We will use the [Scotty starter](https://github.com/scotty-web/scotty-starter) app as a template for the source code.
So fill the github URL `https://github.com/scotty-web/scotty-starter` in the 'Source Code' field. Note that we only use Openshift to host the source and will not be using their resources for building. The built app binary will be uploaded into Openshift and configured to execute.

# Build the app locally

Clone the git repo which is created by Openshift for this app into your local machine. Currently, the Scotty starter uses [Cabal](https://www.haskell.org/cabal) for building, and not the more modern [Stack](https://www.haskellstack.org).
This is easily remedied by doing a `stack init` in the source directory. From now on, I assume the project has been migrated to Stack.

Since we will be building the app locally and uploading the binary to Openshift, the executable will have to be statically linked. Edit `app.cabal` and add this line under the `executable app` section:

      ghc-options:         -optl-static -optl-pthread

Let us tell Stack to store the binary in a place which is easily accessible. Edit `stack.yaml` and add this line:

      local-bin-path: ./bin

This will store the statically linked executable binary in the `bin` sub-directory of the app. The directory should be created before building the app.

Run `stack build` to compile the code and create the executable `./bin/app`. To run the app in Openshift, we can upload this binary as part of the git repo.

# Run the app on Openshift

Openshift uses [action hooks](https://developers.openshift.com/managing-your-applications/action-hooks.html) to launch and kill apps.
Our DIY application (cloned from Scotty starter kit) does not have any hooks right now. Let us add them under `.openshift/action_hooks/start` and `.openshift/action_hooks/stop`.

`.openshift/action_hooks/start` should contain:

    #!/bin/bash
    cd $OPENSHIFT_REPO_DIR/bin
    nohup ./app > ${OPENSHIFT_DIY_LOG_DIR}/binhello.log 2>&1 &

`.openshift/action_hooks/stop` should contain:

    #!/bin/bash
    kill `ps -ef | grep app | grep -v grep | awk '{ print $2 }'` > /dev/null 2>&1
    exit 0

Make these 2 files executable and push them to the Openshift repo. Once the binary and the action hooks are in place, Openshift should be able to start it up. But we still have not told the app to bind to the port and IP that Openshift provides.

Edit `Main.hs` and add this helper function to set the port and IP.

```haskell
-- Set some Scotty settings
opts :: String -> Int -> Options
opts h p = def { verbose = 0
               , settings = setPort p $ setHost (fromString h) $
                            settings def
               }
```

Change the call to `scotty` in `main` to accept these settings:

```haskell
  port <- read <$> getEnv "OPENSHIFT_DIY_PORT"
  host <- getEnv "OPENSHIFT_DIY_IP"
  putStrLn ("Connecting to " ++ host ++ ":" ++ show port)
  scottyOpts (opts host port) $ do ... <rest of code>
```

Re-build the application and push the binary to Openshift. Now, you should be able to see the Scotty starter app running under your-app-name.rhcloud.com.


