# Docker Notes

- Docker runs in 2 modes i.e. attached and detached
    - `start`: This runs the docker in detached mode. This command doesn't console log any thing. Basically this command runs docker in background.
    - `run`: This runs the docker in attached mode. If there is any container which logs some data then this is the best option. Basically this command runs the docker in foreground and it hangs up the terminal.
    
- If container running in detached mode and want to attach then simply run `docker attach :dockerId`.

- If want to interact with docker then pass `interactive flag` in run/start command.

- `ps`: Lists container.

- `images`: Lists images.

- `inspect`: Lists the total description of image.

- `cp`: If you wants to copy files from or to the container. eg. `docker cp source/:filename dockername:/filename`

> Images contain multiple layer. (1 Instruction = 1 layer.). This is used to optimize build speed (caching!) and re-usability.

> Cheat sheets on docker container and images basics is linked [HERE](./assets/docker-basic-cheet-sheets.pdf)

> Note that container is read only. When we build the container, multiple layers are created which is read only. And when we perform any IO related operation. For eg. creating a .txt file and writing something on it. This file is not written on container (read-write layer) which is added on top of read only layer. so any IO related operation(storing, creating file) is done on this read write layer. And once the container is removed this layer is also cleared because it is created when we run a image on container. This is a problem. lets say we want so store some permanent data on text file which we can't afford if we remove the container then?? 😢😢😢

# Volumes

Volumes helps us with persisting the data. 

### Understanding volumes

1. Volumes are folders on your hard-drive/host machine which are mounted or mapped onto your container. 
2. There is a 2 way linking with the volumes and host machine. Which basically means that container can read/write data to and from the volumes. 
3. Volumes persist the data even if the container is removed or stopped.
4. If a container restarts of mounts the volume, any data inside of that volume is available in the container.

### Types of external storages used by docker.

1. Volumes (Managed by docker). - 
    This has 2 types of volumes. Anonymous Volumes and Named Volumes.
2. Mounts (Managed by you).

#### Volumes

In this docker sets up a folder / path to your host machine. exact location is unknown to you (=dev) Managed via `docker volume` commands.

1. **Anonymous Volumes** - The reason why this is called anonymous volumes because the name/id given to this volume is random. And it actually only exists as long as long as our container exists.

```
/** For Anonymous volume. Inside the docker file */

VOLUME [ "/app/sample-volume" ]

// We can also create anonymous volume using command.
$ docker run -v /app/feedback
```

2. **Named Volumes** - The volumes will survive container's shutdown. The folder on your hard drive will also survive and therefore when you start a new container thereafter, the volumes will be back, folder will be back and all the data stored in that folder will still be available. 

```
/** For Named volume. Use terminal */
$ docker run -v feedback:/app/feedback

// Here -v represents the volume flag and syntax is "volume_name:volume_path"
```

**To clear all volumes: $ docker volume rm VOL_NAME**

> Volumes dont really have direct access to read/write the folder on host machine. It is hidden somewhere, managed by docker and not meant to edited by you. This is best if you don't feel need of accessing/editing data directly.  But there is a option!!!

#### Bind mounts

1. This is best while working as a developer where we want to update the snapshot of the local machine code to the running container. 

2. Volumes are managed by docker and we don't really know where it is located on host machine.  But with bind mount, developer set the path to which the container internal path should be mapped on our host machine. So here, we are fully aware of the path on our local machine.

3. Container's cannot just write but also read from there. 

```
/**How to use Bind mounts */
docker run "/Users/kartikchawda/projects/docker-samples/volumes-example-1:/app"

Here in first argument we have to pass the absolute path of the folder/file.

// To make docker read only
docker run -v /app/node_modules:ro
```

> Note: [Mounting Docker volumes with Docker Toolbox for Windows](https://headsigned.com/posts/mounting-docker-volumes-with-docker-toolbox-for-windows/)

> [Accessing file system](./assets/windows-wsl2-file-events.pdf)

> [Slides Volumes](./assets/slides-data-volumes.pdf)

### Containers and Networks.

Docker with networking consists of 3 cases.

1. Communicating to WWW which is not our own domain. It's some other domain.
2. Communicating with local host machine. For example communicating with local mongodb
3. Communicating with Other container

#### Communicating with WWW

Communicating with WWW is automatically managed by the docker.

#### Communicating with local host machine

For communicating with local host machine, we have to replace localhost with special domain.

```
mongoose.connect(
  'mongodb://host.docker.internal:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

**Here `host.docker.internal` acts as a special domain.**

#### Communicating with Other container

Lets assume we run a docker mongodb container and want to connect to that container.

`docker run mongo`

1. Basic solution.

    Look to the IPAddress of running container with `docker container inspect container__name` command. And replace the IP inside the code.

2. Easy Solution.

    Lets assume we have multiple running container. Docker has this functionality where we can all-together put all the containers in a single network using `docker run --network my_network` command. This then creates the network which all together talks to each other and docker automatically doing IP lookup and resolving stuff which we were doing manually in basic solution.

    * First create a network with
    `docker network create network__name`.
    
    * Once the network is created, now run the containers in that network using
    `docker run --network network__name image__name`. Lets consider this is mongodb container running on `abc` network.

    * Now first edit the connection IP Address in source code with the running `container__name`. and then run the build and run the container same like above command.

    * Docker internally does the IP lookup and resolving stuff with this type of communication.

> [Networking PPT](./assets/slides-networking.pdf)

> [Network Requests cheat sheet](./assets/Cheat-Sheet-Networks-Requests.pdf)








