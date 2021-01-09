# Step 1 — Installing Docker
The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. To do that, we’ll add a new package source, add the GPG key from Docker to ensure the downloads are valid, and then install the package.

First, update your existing list of packages:

> sudo apt update

Next, install a few prerequisite packages which let apt use packages over HTTPS:

> sudo apt install apt-transport-https ca-certificates curl software-properties-common

Then add the GPG key for the official Docker repository to your system:

> curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

Add the Docker repository to APT sources:

> sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

Next, update the package database with the Docker packages from the newly added repo:

> sudo apt update

Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:

> apt-cache policy docker-ce

You’ll see output like this, although the version number for Docker may be different:

<pre>
Output of apt-cache policy docker-ce
docker-ce:
  Installed: (none)
  Candidate: 18.03.1~ce~3-0~ubuntu
  Version table:
     18.03.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
</pre>

Notice that docker-ce is not installed, but the candidate for installation is from the Docker repository for Ubuntu 18.04 (bionic).

Finally, install Docker:

> sudo apt install docker-ce

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:

> sudo systemctl status docker

The output should be similar to the following, showing that the service is active and running:

Output:
<pre>
docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-07-05 15:08:39 UTC; 2min 55s ago
     Docs: https://docs.docker.com
 Main PID: 10096 (dockerd)
    Tasks: 16
   CGroup: /system.slice/docker.service
           ├─10096 /usr/bin/dockerd -H fd://
           └─10113 docker-containerd --config /var/run/docker/containerd/containerd.toml
</pre>
Installing Docker now gives you not just the Docker service (daemon) but also the docker command line utility, or the Docker client. We’ll explore how to use the docker command later in this tutorial.

# Step 2 — Executing the Docker Command Without Sudo (Optional)
By default, the docker command can only be run the root user or by a user in the docker group, which is automatically created during Docker’s installation process. If you attempt to run the docker command without prefixing it with sudo or without being in the docker group, you’ll get an output like this:
> docker

Output
<pre>
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
</pre>
If you want to avoid typing sudo whenever you run the docker command, add your username to the docker group:

> sudo usermod -aG docker ${USER}

> sudo usermod -aG docker root

To apply the new group membership, log out of the server and back in, or type the following:

> su - ${USER}

> su - root

You will be prompted to enter your user’s password to continue.

Confirm that your user is now added to the docker group by typing:

> id -nG

Output
<pre>
sammy sudo docker
</pre>
If you need to add a user to the docker group that you’re not logged in as, declare that username explicitly using:

> sudo usermod -aG docker username

The rest of this article assumes you are running the docker command as a user in the docker group. If you choose not to, please prepend the commands with sudo.

Let’s explore the docker command next.

### Step 3 — Using the Docker Command
Using docker consists of passing it a chain of options and commands followed by arguments. The syntax takes this form:

docker [option] [command] [arguments]

To view all available subcommands, type:

> docker


# Step 4 — Installing Docker Compose
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

# Step 5 — Running a Container with Docker Compose
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

# Step 6 — Removing the Image (Optional)
To avoid using unnecessary disk space, we’ll remove the local image. To do so, we’ll need to delete all the containers that reference the image using the docker rm command, followed by either the CONTAINER ID or the NAME. Below, we’re using the CONTAINER ID from the docker ps -a command we just ran. Be sure to substitute the ID of your container:

> docker rm 06069fd5ca23

Once all containers that reference the image have been removed, we can remove the image:

> docker rmi hello-world


































# Step 7 — Deploying nginx-proxy with Let’s Encrypt
In this section, you’ll deploy nginx-proxy and its Let’s Encrypt add-on using Docker Compose. This will enable automatic TLS certificate provisioning and renewal, so that when you deploy Eclipse Theia it will be accessible at your domain via HTTPS.

For the purposes of this tutorial, you’ll store all files under ~/eclipse-theia. Create the directory by running the following command:

> mkdir ~/eclipse-theia

Navigate to it:

> cd ~/eclipse-theia

You’ll store the Docker Compose configuration for nginx-proxy in a file named nginx-proxy-compose.yaml. Create it using your text editor:

> nano nginx-proxy-compose.yaml

Add the following lines:

~/eclipse-theia/nginx-proxy-compose.yaml

<pre>
version: '2'

services:
  nginx-proxy:
    restart: always
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/etc/nginx/htpasswd:/etc/nginx/htpasswd"
      - "/etc/nginx/vhost.d"
      - "/usr/share/nginx/html"
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
      - "/etc/nginx/certs"

  letsencrypt-nginx-proxy-companion:
    restart: always
    image: jrcs/letsencrypt-nginx-proxy-companion
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    volumes_from:
      - "nginx-proxy"
</pre>

Here you’re defining two services that Docker Compose will run, nginx-proxy and its Let’s Encrypt companion. For the proxy, you specify jwilder/nginx-proxy as the image, map HTTP and HTTPS ports, and define volumes that will be accessible to it during runtime.

Volumes are directories on your server that the defined service will have full access to, which you’ll later use to set up user authentication. To achieve that, you’ll make use of the first volume from the list, which maps the local /etc/nginx/htpasswd directory to the same one in the container. In that folder, nginx-proxy expects to find a file named exactly as the target domain, containing log-in credentials for user authentication in the htpasswd format (username:hashed_password).

For the add-on, you name the Docker image and allow access to Docker’s socket by defining a volume. Then, you specify that the add-on should inherit access to the volumes defined for nginx-proxy. Both services have restart set to always, which orders Docker to restart the containers in case of crash or system reboot.

Save and close the file.

Deploy the configuration by running:

> docker-compose -f nginx-proxy-compose.yaml up -d

Here you pass in the nginx-proxy-compose.yaml filename to the -f parameter of the docker-compose command, which specifies the file to run. Then, you pass the up verb that instructs it to run the containers. The -d flag enables detached mode, which means that Docker Compose will run the containers in the background.

The final output will look like this:

Output<pre>
Creating network "eclipse-theia_default" with the default driver
Pulling nginx-proxy (jwilder/nginx-proxy:)...
latest: Pulling from jwilder/nginx-proxy
8d691f585fa8: Pull complete
5b07f4e08ad0: Pull complete
...
Digest: sha256:dfc0666b9747a6fc851f5fb9b03e65e957b34c95d9635b4b5d1d6b01104bde28
Status: Downloaded newer image for jwilder/nginx-proxy:latest
Pulling letsencrypt-nginx-proxy-companion (jrcs/letsencrypt-nginx-proxy-companion:)...
latest: Pulling from jrcs/letsencrypt-nginx-proxy-companion
89d9c30c1d48: Pull complete
668840c175f8: Pull complete
...
Digest: sha256:a8d369d84079a923fdec8ce2f85827917a15022b0dae9be73e6a0db03be95b5a
Status: Downloaded newer image for jrcs/letsencrypt-nginx-proxy-companion:latest
Creating eclipse-theia_nginx-proxy_1 ... done
Creating eclipse-theia_letsencrypt-nginx-proxy-companion_1 ... done
</pre>
You’ve deployed nginx-proxy and its Let’s Encrypt companion using Docker Compose. Now you’ll move on to set up Eclipse Theia at your domain and secure it.

# Step 8 — Deploying Dockerized Eclipse Theia
In this section, you’ll create a file containing any allowed log-in combinations that a user will need to input. Then, you’ll deploy Eclipse Theia to your server using Docker Compose and expose it at your secured domain using nginx-proxy.

As explained in the previous step, nginx-proxy expects log-in combinations to be in a file named after the exposed domain, in the htpasswd format and stored under the /etc/nginx/htpasswd directory in the container. The local directory which maps to the virtual one does not need to be the same, as was specified in the nginx-proxy config.

To create log-in combinations, you’ll first need to install htpasswd by running the following command:

> sudo apt install apache2-utils

The apache2-utils package contains the htpasswd utility.

Create the /etc/nginx/htpasswd directory:

> sudo mkdir -p /etc/nginx/htpasswd

Create a file that will store the logins for your domain:

> sudo touch /etc/nginx/htpasswd/theia.your_domain

Remember to replace theia.your_domain with your Eclipse Theia domain.

To add a username and password combination, run the following command:

> sudo htpasswd /etc/nginx/htpasswd/theia.your_domain username

Replace username with the username you want to add. You’ll be asked for a password twice. After providing it, htpasswd will add the username and hashed password pair to the end of the file. You can repeat this command for as many logins as you wish to add.

Now, you’ll create configuration for deploying Eclipse Theia. You’ll store it in a file named eclipse-theia-compose.yaml. Create it using your text editor:

> nano eclipse-theia-compose.yaml

Add the following lines:

~/eclipse-theia/eclipse-theia-compose.yaml
<pre>
version: '2.2'

services:
  eclipse-theia:
    restart: always
    image: theiaide/theia:next
    init: true
    environment:
      - VIRTUAL_HOST=theia.your_domain
      - LETSENCRYPT_HOST=theia.your_domain
</pre>
In this config, you define a single service called eclipse-theia with restart set to always and theiaide/theia:next as the container image. You also set init to true to instruct Docker to use init as the main process manager when running Eclipse Theia inside the container.

Then, you specify two environment variables in the environment section: VIRTUAL_HOST and LETSENCRYPT_HOST. The former is passed on to nginx-proxy and tells it at what domain the container should be exposed, while the latter is used by its Let’s Encrypt add-on and specifies for which domain to request TLS certificates. Unless you specify a wildcard as the value for VIRTUAL_HOST, they must be the same.

Remember to replace theia.your_domain with your desired domain, then save and close the file.

Now deploy Eclipse Theia by running:

> docker-compose -f eclipse-theia-compose.yaml up -d

The final output will look like:

Output
<pre>
...
Pulling eclipse-theia (theiaide/theia:next)...
next: Pulling from theiaide/theia
63bc94deeb28: Pull complete
100db3e2539d: Pull complete
...
Digest: sha256:c36dff04e250f1ac52d13f6d6e15ab3e9b8cad9ad68aba0208312e0788ecb109
Status: Downloaded newer image for theiaide/theia:next
Creating eclipse-theia_eclipse-theia_1 ... done
</pre>
Then, in your browser, navigate to the domain you’re using for Eclipse Theia. Your browser will show you a prompt asking you to log in. After providing the correct credentials, you’ll enter Eclipse Theia and immediately see its editor GUI. In your address bar you’ll see a padlock indicating that the connection is secure. If you don’t see this immediately, wait a few minutes for the TLS certificates to provision, then reload the page.

Eclipse Theia GUI

![Dockar Hub](https://github.com/bipulcn/importdoc/blob/main/theia/img/img1.png)

Now that you can securely access your cloud IDE, you’ll start using the editor in the next step.

# Step 9 — Using the Eclipse Theia Interface
In this section, you’ll explore some of the features of the Eclipse Theia interface.

On the left-hand side of the IDE, there is a vertical row of four buttons opening the most commonly used features in a side panel.

Eclipse Theia GUI - Sidepanel
![Eclipse Theia GUI - Sidepanel](https://github.com/bipulcn/importdoc/blob/main/theia/img/img2.png)

This bar is customizable so you can move these views to a different order or remove them from the bar. By default, the first view opens the Explorer panel that provides tree-like navigation of the project’s structure. You can manage your folders and files here—creating, deleting, moving, and renaming them as necessary.

After creating a new file through the File menu, you’ll see an empty file open in a new tab. Once saved, you can view the file’s name in the Explorer side panel. To create folders right click on the Explorer sidebar and click on New Folder. You can expand a folder by clicking on its name as well as dragging and dropping files and folders to upper parts of the hierarchy to move them to a new location.

Eclipse Theia GUI - New Folder
![Eclipse Theia GUI - New Folder](https://github.com/bipulcn/importdoc/blob/main/theia/img/img3.png)

The next two options provide access to search and replace functionality. Following it, the next one provides a view of source control systems that you may be using, such as Git.

The final view is the debugger option, which provides all the common actions for debugging in the panel. You can save debugging configurations in the launch.json file.

Debugger View with launch.json open
![Theia Hub](https://github.com/bipulcn/importdoc/blob/main/theia/img/img4.png)

The central part of the GUI is your editor, which you can separate by tabs for your code editing. You can change your editing view to a grid system or to side-by-side files. Like all modern IDEs, Eclipse Theia supports syntax highlighting for your code.

Editor Grid View
![Editor Grid View](https://github.com/bipulcn/importdoc/blob/main/theia/img/img5.png)

You can gain access to a terminal by typing CTRL+SHIFT+, or by clicking on Terminal in the upper menu, and selecting New Terminal. The terminal will open in a lower panel and its working directory will be set to the project’s workspace, which contains the files and folders shown in the Explorer side panel.

Terminal open
![Terminal open](https://github.com/bipulcn/importdoc/blob/main/theia/img/img6.png)

You’ve explored a high-level overview of the Eclipse Theia interface and reviewed some of the most commonly used features.

[resource 1](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
[resource 2](https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04)
[resource 3](https://www.digitalocean.com/community/tutorials/how-to-set-up-the-eclipse-theia-cloud-ide-platform-on-ubuntu-18-04)
