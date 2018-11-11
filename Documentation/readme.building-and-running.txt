1) Running solution in Visual Studio

Greate article and video about containerization in Visual Studio 2017:
https://www.scrum-tips.com/2017/12/27/understanding-docker-with-visual-studio-2017-part-2/
https://app.pluralsight.com/player?course=docker-visual-studio-2017-windows&author=marcel-devries&name=docker-visual-studio-2017-windows-m2&clip=3&mode=live

Before application will be run (and debugged) it must be built and build process differs depending on build mode.
In release mode application is build and output is published to "obj/Docker/publish", also additional docker-compose file is created in this mode ("docker-compose.vs.release.g.yml"). In general this file (with addition to other docker-compose files) is used by Visual Studio which runs docker-compose up command. Release docker compose file is responsible for setting "source" build argument to the same folder. This source folder is used in dockerfile to copy artifacts to container. VS remote debugger is also additional container volume to which container will be attached after being started.
In debug mode application is build and output is also published to "obj/Docker/publish", also appropriate docker-compile file is created in this mode ("docker-compose.vs.debug"g.yml)". This file has different purpose than the one in release mode. It doesn't set "source" argument, which causes that instead of copying artifacts to container the direct volume with application is shared with host and container. After all, like in release, remote debugger is attached to run application.

So in general, under the hood using "Start debugging" (F5), Visual studio runs command:
docker-compose -f "docker-compose.yml" -f "docker-compose.override.yml" -f "obj\Docker\docker-compose.vs.{debug/release}.g.yml" -p someRandomNameProjectName up -d --build 

1.1) Applications with frontend client application

Normally web application handles angular client application in two ways depending on build target:
- in Development, application is served by separated angular-cli server, so all angular-cli prerequisites need to be preinstalled on container (node, npm, node-sass and its bindings)
- in Production, application is served by static, precompiled files from obj/Docker/publish/ClientApp/dist/ (node is not needed in container in that case)

You can determine build target by checking environment variables in docker-compose file for specific container.

Running application in Prodution build target is a little bit tricky because it need precompiled and published client application (see point 1.1) at "obj/Docker/publish" (see Dockerfile copy operation).
Running application in Development build target is also a little bit tricky because requires that container have installed all angular prerequisites, especially binding of node-sass.
Recompiling bindings of node-sass is required because you normally use windows and node-sass have binding for windows, but in container you need bindings for linux, that is why running application in development mode breaks it.
When incorrect binding are detected, go to point 1.2.

1.1) Building application

`dotnet build -c {Debug|Release}` - builds just dotnet application
`dotnet publish -c {Debug|Release}` - builds dotnet application, but also builds client application

1.2) Incorrect node-sass binding

Sometimes if you switch working platforms (linux/windows) you can reach situation of incorrect node-sass bindings. The best way is to have installed both version of binding.
In case of situation where there is a lack of linux binding, try to attach to running container by:

docker exec -it _hash_ /bin/bash

and then

npm rebuild node-sass --force

After all, stop container and start it again. Staring angular-cli development server should be broken now.

Best usage:

dotnet publish -c Release -o obj\Docker\publish

2) Running solution manually with docker outside Visual Studio 2017

You can run solution without remote debugging in Visual Studio 2017 by:

docker-compose -f .\docker-compose.yml -f .\docker-compose.override.yml up

There are a few issues with running solution without Visual Studio which are described in next sections

2.1) Problem with local development https certificate in production target

There is an issue with starting Kestrel in production when https is used. Kestrel by defualt try to find development certificates in wrong place, so in another docker-compose file we put special environment variables for that

docker-compose -f .\docker-compose.yml -f .\docker-compose.override.yml -f .\docker-compose.override.production.yml up

2.2) Problem with angular cli server for development target when running outside Visual Studio

Problem occurs because in Startup.cs we assume that when application is run in development then angular cli server is run instead of serving static files. Angular cli server need full client application code with is copied to container when running dockerized application with Visual Studio. Additional docker-compose file is prepared for that case which sets appropariate environment variable for blocking angular cli server even in development.

docker-compose -f .\docker-compose.yml -f .\docker-compose.override.yml -f .\docker-compose.override.development.yml up


2.2) Dockerizing standalone mode

docker-compose -f .\docker-compose.yml -f .\docker-compose.production.yml up

2.3)

docker-compose -f .\docker-compose.yml -f .\docker-compose.production.yml -f obj\Docker\docker-compose.vs.{debug/release}.g.yml up

3) Helpers

- stop all container (powershell)
docker stop $(docker ps -aq)