# Introduction to Docker {#docker-intro status=beta}

<img src="pics/docker_logo.png" id="docker_logo"/>

<style>
#docker_logo {
width: 10em;
}
figure img {
max-width: 100%;
}
</style>

## Docker is a tool for portable, reproducible computing

It would be nice to give a computer - any computer with an internet connection - a short string of ASCII characters (say via a keyboard), press enter, and return to see some program running. Forget about where the program was built or what software you happened to be running at the time (this can be checked, and we can fetch the necessary dependencies). Sounds simple, right? In fact, this is an engineering task that has taken thousands of the world's brightest developers many decades to implement.

Thanks to the magic of [container technology](https://en.wikipedia.org/wiki/Operating-system-level_virtualization) we now can run any Linux program on almost any networked device on the planet, as is. All of the environment preparation, installation and configuration steps can be automated from start to finish. Depending on how much network bandwidth you have, it might take a while, but that's all right. All you need to do is type the string correctly.

## Docker containers are easy to install

Let's say you have never used Docker. To get Docker, run this command on a POSIX shell of any [Docker-supported platform](https://docs.docker.com/install/#supported-platforms):

```
$ curl -sSL https://get.docker.com/ | sh
```

Now you have installed Docker! 

Suppose your friend, Daphne, has a Docker **container**. How can we run this container? Docker containers live inside **registries**, which are servers that host Docker images. A Docker **image** is basically a filesystem snapshot---a single file that contains everything you need to run her container.

<figure id="docker_registry" markdown="1">
<img src="pics/docker_registry.png"/>
<figcaption>
Docker ships with a default registry, called the <a href="https://hub.docker.com">Docker Hub</a>, a big server that is home to many useful repositories.
</figcaption>
</figure>

You can fetch Daphne's container by running the following command to pull it from her Docker Hub repository:

```
$ docker pull daphne/duck
```

Now you have Daphne's Docker image. To see a list of Docker images on your machine, type:

```
$ docker images
```

Every image has a image ID, a name and a tag:

```
REPOSITORY      TAG        IMAGE ID         CREATED       SIZE
daphne/duck     latest     ea2f90g8de9e     1 day ago     869MB
```

To run a Docker container, type the name of the container, like so:

```
$ docker run daphne/duck
```

Now you are running Daphne's container. To verify it is running, type:

```
$ docker ps
CONTAINER ID     IMAGE           ...     NAMES
52994ef22481     daphne/duck     ...     happy_hamster
```

Note how Daphne's duck container has a *container ID*, a base image, and a funny-looking name, `happy_hamster`. You can use this name as an alias for the container ID.

## Containers come from other containers

So you have a terminal and an internet connection? Now it doesn't matter what operating system you're running! You can run almost any Linux program in the world with just a few keystrokes. No further steps are necessary. How neat is that? To have a tool that clones a program and its environment, fetches the appropriate dependencies, and runs it on any OS is a big time-saver. Suppose you have a program that runs on one computer. It is extremely likely to run on any other, regardless of the underlying OS or hardware.

But how do you create a Docker **image**? There are two ways. You can either snapshot a running Docker container, or you can write a plaintext recipe. First, let's see how to create a snapshot:

```
$ docker run -it daphne/duck bash
```

This will launch Daphne's container and drop us into a bash session within. Suppose we make a change to the Docker container like so:


    root@295fd7879184:/# touch new_file && ls
    total 0
    -rw-r--r-- 1 root root 0 May 21 20:52 new_file


However, this file is not permanent. Exit the container:

    root@295fd7879184:/# exit

Then rerun it:

    $ docker run -it daphne/duck bash

And run:

    root@18f13bb4571a:/# ls
    root@18f13bb4571a:/# touch new_file1 && ls -l
    total 0
    -rw-r--r-- 1 root root 0 May 21 21:32 new_file1

It seems like `new_file` has disappeared! Notice how the container ID (`18f13bb4571a`) is now different. This is because `docker run daphne/duck` created a new container from the image `daphne/duck`, rather than restarting our old container. 

Let's see all the containers on our machine:

```
$ docker container ls -a
CONTAINER ID     IMAGE           ...     STATUS                              NAMES
295fd7879184     daphne/duck     ...     Exited (130) About a minute ago     merry_manatee
18f13bb4571a     daphne/duck     ...     Up 5 minutes                        shady_giraffe
52994ef22481     daphne/duck     ...     Up 10 minutes                       happy_hamster
```

It looks like `295fd7879184` a.k.a. `merry_manatee` survived, but it is no longer running. Whenever a container's main process (recall we ran `merry_manatee` with `bash`) finishes, the container will stop, but it will not cease to exist.

In fact, we can resume the stopped container right where we left off:

```
$ docker start -a merry_manatee
root@295fd7879184:/# ls -l
total 0
-rw-r--r-- 1 root root 0 May 21 20:52 new_file
```

Nothing was lost! Let's open a new terminal (without leaving the current one) to see what other containers are running:

```
$ docker ps
CONTAINER ID     IMAGE           ...     NAMES
295fd7879184     daphne/duck     ...     merry_manatee
18f13bb4571a     daphne/duck     ...     shady_giraffe
52994ef22481     daphne/duck     ...     happy_hamster
```

Now suppose we would like to share the container `shady_giraffe` with someone else. To do so, we must first snapshot the running container, or **commit** it to a new image, giving it a name and a tag. This will create a checkpoint that we may later restore:

```
$ docker commit -m "fork Daphne's duck" shady_giraffe your/duck:v2
```

Wherever you see a funny-looking name like `shady_giraffe` in Docker, this is just another way to reference the container. We either can use the container ID, `18f13bb4571a` or the designated name (ie. `shady_giraffe`). The above `your` can be your username or an organization you belong to on a Docker registry. This image will be called `your/duck`, and has an optional version identifier, `v2`. Now we can push it to the registry:

```
$ docker push your/duck:v2
$ docker images
REPOSITORY    TAG        IMAGE ID         CREATED          SIZE
daphne/duck   latest     ea2f90g8de9e     1 day ago        869MB
your/duck     v2         d78be5cf073e     2 seconds ago    869MB
$ docker pull your/duck:v2 # Anyone can run this!
$ docker run your/duck ls -l
total 0
-rw-r--r-- 1 root root 0 May 21 21:32 new_file1
```

This is a convenient way to share an image with colleagues and collaborators. Anyone with access to the repository can pull our image and continue right where we left off, or create another image based on our own. Images can be created via the command line or by using something called a `Dockerfile`.

## Containers come from recipes

The second way to create a Docker image is to write a recipe, called a `Dockerfile`. A `Dockerfile` is a text file that specifies the commands required to create a Docker image, typically by modifying an existing container image using a scripting interface. They also have [special keywords](https://docs.docker.com/engine/reference/builder) (which are always CAPITALIZED), like [`FROM`](https://docs.docker.com/engine/reference/builder/#from), [`RUN`](https://docs.docker.com/engine/reference/builder/#run), [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint) and so on. For example:

```
$ echo -e '
FROM dapne/duck
RUN touch new_file1   # new_file1 will be part of our snapshot
CMD ls -l             # Default command to be run when the container is started
' >> Dockerfile
```

Now, to build the image we can simply run:

```
$ docker build -t your/duck:v3 . # Where '.' is the directory containing your Dockerfile
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM daphne/duck
 ---> ea2f90g8de9e
Step 2/3 : RUN touch new_file1
 ---> e3b75gt9zyc4
Step 3/3 : CMD ls -l
 ---> Running in 14f834yud59
Removing intermediate container 14f834yud59
 ---> 05a3bd381fc2
Successfully built 05a3bd381fc2
Successfully tagged your/duck:v3
$ docker images
REPOSITORY    TAG        IMAGE ID         CREATED          SIZE
daphne/duck   latest     ea2f90g8de9e     1 day ago        869MB
your/duck     v2         d78be5cf073e     5 minutes ago    869MB
your/duck     v3         05a3bd381fc2     2 seconds ago    869MB
```

This procedure is identical to the snapshot method we performed earlier, except the result is much cleaner. Now, instead of needing to carry around a 869MB BLOB, we can just store the 4KB text file and may rest assured that all our important setup commands are contained within. Similar to before, we can simply run:

```
$ docker run -it your/duck:v3
total 0
-rw-r--r-- 1 root root 0 May 21 21:35 new_file1
```

Notice that as soon as we run the container, Docker will execute the `ls -l` command as specified by the `Dockerfile`, revealing `new_file1` was stored in the image. However we can still override `ls -l` by passing a command line argument: `docker run -it your/duck:v3 [custom_command]`.

Docker uses a concept of [*layers*](https://docs.docker.com/storage/storagedriver/#images-and-layers). Every instruction we add to the Dockerfile beginning with a Dockerfile keyword will add a new layer, which is conveniently cached by the [Docker daemon](https://docs.docker.com/engine/reference/commandline/dockerd/). If we should modify a Dockerfile, Docker will only need to rebuild the image starting from the first modified instruction. Let's have a look:

    FROM dapne/duck                             # Defines the base container
    RUN touch new_file1                         # Defines a new layer
    RUN mkdir config && mv new_file1 mkdir      # Each layer can have multiple commands
    RUN curl -sSL https://get.your.app/ | sh    # Layers can have a script

Suppose we make a change at the bottom of our Dockerfile. If Docker had to rerun the entire recipe from top to bottom to every time we wanted to build the image, this would be terribly slow and inconvenient. Fortunately, Docker is smart enough to cache the layers which have not changed, and only rerun the minimum set of commands to rebuild our image. This is a very nice feature, however it can sometimes introduce unexpected results, especially when the cache is stale. To ignore the cache and force a clean rebuild of a docker image, use `docker build --no-cache`.

We can also chain `Dockerfile`s together using a technique called [*multi-stage builds*](https://docs.docker.com/develop/develop-images/multistage-build/). These allow you to build multiple images into one `Dockerfile`, and copy resources from one image to another:

    FROM your/duck:v3 as template1              # We can use `template1` as an alias later

    FROM daphne/duck as template2
    COPY --from=template1 new_file1 new_file2

    FROM donald/duck as template3               # The last `FROM` will define *this* image
    COPY --from=template2 new_file2 new_file3
    CMD ls -l

Now let's build and run this image:

```
$ docker build . -t your/duck:v4 
Sending build context to Docker daemon  2.048kB
Step 1/6 : FROM your/duck:v3 as template1
 ---> e3b75ef8ecc4
Step 2/6 : FROM daphne/duck as template2
 ---> ea2f90g8de9e
Step 3/6 : COPY --from=template1 new_file1 new_file2
 ---> 72b96668378e
Step 4/6 : FROM donald/duck:v3 as template3
 ---> e3b75ef8ecc4
Step 5/6 : COPY --from=template2 new_file2 new_file3
 ---> cb1b84277228
Step 6/6 : CMD ls
 ---> Running in cb1b84277228
Removing intermediate container cb1b84277228
 ---> c7dc5dd63e77
Successfully built c7dc5dd63e77
Successfully tagged your/duck:v4
$ docker run -it your/duck:v4
total 0
-rw-r--r-- 1 root root 0 Jul  8 15:06 new_file3
```

Why would you want this feature? For example, one application of multi-stage builds, might be compiling a dependency from its source code. In addition to all the source code, the compilation itself has separate build dependencies and setup requirements. All of this is quite superfluous to our end goal, which is to build a single file. If we're unlucky, we might have gigabytes of transitive dependencies to build our tiny image, and spend a lot of effort to clean up the mess afterwards. With multi-stage builds, we can build the file, discard the intermediate layers and move on with our life, unburdened by intermediate dependencies.
