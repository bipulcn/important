# Step 1 — Installing Docker Compose
Although we can install Docker Compose from the official Ubuntu repositories, it is several minor version behind the latest release, so we’ll install Docker Compose from the Docker’s GitHub repository. The command below is slightly different than the one you’ll find on the Releases page. By using the -o flag to specify the output file first rather than redirecting the output, this syntax avoids running into a permission denied error caused when using sudo.

We’ll check the current release and if necessary, update it in the command below:

> sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

Next we’ll set the permissions:

> sudo chmod +x /usr/local/bin/docker-compose

Then we’ll verify that the installation was successful by checking the version:

> docker-compose --version

This will print out the version we installed:

Output
<pre>
docker-compose version 1.21.2, build a133471
</pre>
Now that we have Docker Compose installed, we’re ready to run a “Hello World” example.

# Step 2 — Running a Container with Docker Compose
The public Docker registry, Docker Hub, includes a Hello World image for demonstration and testing. It illustrates the minimal configuration required to run a container using Docker Compose: a YAML file that calls a single image:

First, we’ll create a directory for the YAML file and move into it:

> mkdir hello-world
> cd hello-world

Then, we’ll create the YAML file:

> nano docker-compose.yml

Put the following contents into the file, save the file, and exit the text editor:

docker-compose.yml

<pre>
my-test:
 image: hello-world
</pre>

The first line in the YAML file is used as part of the container name. The second line specifies which image to use to create the container. When we run the command docker-compose up it will look for a local image by the name we specified, hello-world. With this in place, we’ll save and exit the file.

We can look manually at images on our system with the docker images command:

> docker images

When there are no local images at all, only the column headings display:

Output<pre>
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
</pre>

Now, while still in the ~/hello-world directory, we’ll execute the following command:

> docker-compose up

The first time we run the command, if there’s no local image named hello-world, Docker Compose will pull it from the Docker Hub public repository:

Output<pre>
Pulling my-test (hello-world:latest)...
latest: Pulling from library/hello-world
c04b14da8d14: Downloading [==================================================>] c04b14da8d14: Extracting [==================================================>]  c04b14da8d14: Extracting [==================================================>]  c04b14da8d14: Pull complete
Digest: sha256:0256e8a36e2070f7bf2d0b0763dbabdd67798512411de4cdcf9431a1feb60fd9
Status: Downloaded newer image for hello-world:latest
. . .
</pre>
After pulling the image, docker-compose creates a container, attaches, and runs the hello program, which in turn confirms that the installation appears to be working:

Output<pre>
. . .
Creating helloworld_my-test_1...
Attaching to helloworld_my-test_1
my-test_1 |
my-test_1 | Hello from Docker.
my-test_1 | This message shows that your installation appears to be working correctly.
my-test_1 |
. . .</pre>
Then it prints an explanation of what it did:

Output of docker-compose up
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
3. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.

Docker containers only run as long as the command is active, so once hello finished running, the container stopped. Consequently, when we look at active processes, the column headers will appear, but the hello-world container won’t be listed because it’s not running.

> docker ps

Output<pre>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
</pre>
We can see the container information, which we’ll need in the next step, by using the -a flag which shows all containers, not just the active ones:

> docker ps -a

Output<pre>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
06069fd5ca23        hello-world         "/hello"            35 minutes ago      Exited (0) 35 minutes ago                       drunk_payne
</pre>
This displays the information we’ll need to remove the container when we’re done with it.

# Step 3 — Removing the Image (Optional)
To avoid using unnecessary disk space, we’ll remove the local image. To do so, we’ll need to delete all the containers that reference the image using the docker rm command, followed by either the CONTAINER ID or the NAME. Below, we’re using the CONTAINER ID from the docker ps -a command we just ran. Be sure to substitute the ID of your container:

> docker rm 06069fd5ca23

Once all containers that reference the image have been removed, we can remove the image:

> docker rmi hello-world


[resource 2](https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04)
