# About this Repo

Recently, I was trying to develop .NET Core applications in Linux environment without
installing its runtime and SDK. I've used the guidelines in the 
[official repository](https://github.com/dotnet/dotnet-docker/blob/master/samples/dotnetapp/dotnet-docker-dev-in-container.md). It works good, but there is a subtle problem
when you need to save directory or files generated from container into the host: file 
permissions. 

[.NET Core official images](https://hub.docker.com/_/microsoft-dotnet-core) run as 
root user so all generated files belong to _"root:root"_. Therefore, when you try
to modify any of the created files in the host you get a _"permission denied"_ 
message (see [issue 271](https://github.com/aspnet/aspnet-docker/issues/271)). 
In the case you want to have full control of created files in the host
you must run the container with the same user you want to be the real owner.

One may find `--user` option in `docker run` command a valid choice to  solve this
problem, but if the user entered isn't registered in the container, it won't work.
Check below:

![Docker run image]
(imgs/docker_run_01.png)



the images described in this repository assure the user and group exist in the 
.NET core SDK container.

# Previous readings and requirements

You should read the guidelines in the repository for 
[.NET Core Docker images](https://github.com/dotnet/dotnet-docker). Particularly,
you should understand the development model described in 
[Develop .NET Core Applications in a Container](https://github.com/dotnet/dotnet-docker/blob/master/samples/dotnetapp/dotnet-docker-dev-in-container.md). 

# Image List

# Usage

The Dockerfiles in this  repo relies on build-time variables as parameters to determine
the user in the container.

You should build a custom image passing the _"arg-variables"_ `URS`, `UID`, `GRP` 
and `GID`.

```shell
docker build --build-arg USR=$(id -nu) --build-arg UID=$(id -u) --build-arg GRP=$(id -ng) --build-arg GID=$(id -g) -t my-dotnetcore-sdk -f Dockerfile-dotnetcore-sdk .
```

Thereafter, you can use the new image just like the following example:

```shell
docker run --rm -v ~/devel/apis:/home/$(id -nu) my-dotnetcore-sdk dotnet new webapi -o MyApi
```