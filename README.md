# TOR Service Composer

Use this compose setup to generate a Tor hidden service with Nginx serving static assets. Simply replace the contents of the folder `your-project` with your static assets. Your `index.html` file should be located in `tor-composer/your-project/index.html`.

> [!NOTE]  
> This project has an included html website that is meant to help understand how to organize your project and test initial functionality of the tor hidden-service. If you are having trouble use this to help troubleshoot.

## Prerequisites

For this project you will need to have docker with docker compose installed on the machine you plan to host on. Docker is available for Windows, MacOS, Linux distros, and ARM based architechtures.

Go to [Docker website](https://www.docker.com/products/docker-desktop/) to download the needed software. Docker is free for personal use at the time of this post. If you intend to use this for business, it will require a license purchase.

# Operational Security (OpSec)

Using docker to protect your identity is already a great choice. The isolation that this provides your host machine is ideal for hosting hidden-services. You can remain safe so long as you follow this practical advice.

> [!CAUTION]  
> Please be advised this project does not require port forwarding.**If the machine you are running docker on can reach _http and https websites_, nothing is needed to be done.** To maintain isolation and network security please adhere to the following best practices below:
>
> - **DO _NOT_ forward any ports on your network!** This is not necessary to host onion services.
>
> - **DO _NOT_ open ports in the docker-compose file.** This is not recommended and could cause docker to no longer isolate the containers from the host.

## Meta Data

Some may be concerned with leaving traceable data available in the browser for users to examine. This could lead to someone discovering idenitifiable information about you. You should remove all meta data from images before hosting them on your onion site.

# Setup Instructions

Begin by cloning this repository. Open the terminal or command prompt and navigate to the root of the cloned repo. This is where we will issue the docker commands.

> [!NOTE]  
> You will use the [your-project](/your-project/) directory to host serve your website. If you already have a website built, simply delete the contents of `your-project`. If you are simply testing this project out, you can proceed and deploy the included site.

## Step One

Copy your website files and folders into the `your-project` directory.
Once you have added all your static web files to [your-project](/your-project/) (ie. html,css, javascipt, images), in the terminal, at the root of `tor-composer` project, issue the following command:

```bash
docker-compose up --build
```

## Step Two

Once you see that the build is complete and both containers are running, you can enter the container with the following command:

```bash
docker exec -it tor /bin/sh
```

## Step Three

Inside the tor container, issue the following command to print your .onion address:

```bash
cat /var/lib/tor/.tor/hostname
```

This will print your .onion address to the terminal. Open the Tor Browser and navigate to your newly hosted site.

## Bind Mounts

This project does not need to be restarted to update changes to the website. Changes are being tracked in the `your-project` folder. If you modify files in the `your-project` directory while the services are running, the changes will happen immediately.

For this reason you should develop your website outside of this folder, then copy the whole project over when making changes. This way you don't break your site while developing.

## Testing TOR

The tor service in this project, by default, is configured to proxy traffic on localhost:9050. You can test that tor is working by using the curl command. First you will need to enter the tor service container.

In the terminal, navigate to the root of tor-composer and issue the following command:

```bash
docker exec -it tor /bin/sh
```

Now to test the tor service, inside the container issue the following command

```bash
curl --socks5 localhost:9050 --socks5-hostname localhost:9050 https://check.torproject.org/api/ip
```

If tor is configured properly, you will return the following, where the X's represent your exit IP. This should be different than your ISP provided IP address.

```bash
{"IsTor":true,"IP":"XXX.XXX.XXX.XXX"}
```

If `"IsTor:"` returns `false`, you are not using tor and will need to troubleshoot further.

> [!NOTE]  
> Possible issues that prevent you from returning a true response
>
> - Network configurations preventing tor traffic
> - Host firewalling preventing docker from accessing network
> - Not being connected to the internet

> [!IMPORTANT]  
> Back-up Your Data
> It would be a good idea to backup your Docker volumes to avoid losing your hidden-service keys and address. If you are using docker desktop you will need to login to backup or migrate your volumes.

# Adding Your Own Key Files

If you already have an .onion address and would like to use it, you may load your keys into the container using the following method. Before you start the container, move the folder containing the keys, address, and authorized_clients into the projects `hiddenservices` directory.

## Adding Keys to Project

If the folder that your are moving to the project is not currently named `hidden-services`, rename it before you run the docker compose command. In the docker-compose.yml file you will need to uncomment the line below the comment:

```yaml
volumes:
  - ./torrc.template:/etc/tor/torrc.template
  - torkeys:/var/lib/tor/
  - ngxaccesslog:/var/log/tor/host.access.log
  # Remove comment from the line below to add key files
  # - ./hiddenservices:/usr/tor/home/
restart: always
```

Remove the comment like this:

```yaml
volumes:
  - ./torrc.template:/etc/tor/torrc.template
  - torkeys:/var/lib/tor/
  - ngxaccesslog:/var/log/tor/host.access.log
  # Remove comment from the line below to add key files
  - ./hiddenservices:/usr/tor/home/
restart: always
```

## Assigning Permissions

We will need to enter the container as root and assign some permissions to the folder so that we can copy our keys into the `HiddenServiceDir`. In the terminal navigate to the root of the tor-composer project. You will need to start the containers with

```bash
docker-compose up --build
```

Once the container has started type or paste the following command to enter the container.

```bash
docker exec -it tor /bin/sh
```

Now that you are in the container check to make sure your key files are copied and what user owns them with the following

```bash
ls -l /usr/tor/home/hidden-services
```

which should give you an output that looks something like this

```bash
total 0
-rwx------  1 tor tor  63 Feb 26 00:36 hostname
-rwx------  1 tor tor  64 Feb 26 00:36 hs_ed25519_public_key
-rwx------  1 tor tor  96 Feb 26 00:36 hs_ed25519_secret_key
```

Once you know your files are in the container you can use the following commands to copy the data and print the hostname to the terminal

```bash
cp /usr/tor/home/hidden-services/* /var/lib/tor/.tor
cat /var/lib/tor/.tor/hostname
```

## Clean-up

If after running the last command you see your onion address printed to the terminal you can now exit the container with ctrl+d or by typing exit and hitting enter. After exiting container you will

```bash
docker-compose down
```

Go back to the docker-compose.yml file and comment the line we uncommented. It should look like the example below when done.

```yaml
volumes:
  - ./torrc.template:/etc/tor/torrc.template
  - torkeys:/var/lib/tor/
  - ngxaccesslog:/var/log/tor/host.access.log
  # Remove comment from the line below to add key files
  # - ./hiddenservices:/usr/tor/home/
restart: always
```

## Rebuild Project

Finally you can go ahead in the root of tor-composer project and build the project again.

```bash
docker-compose up --build
```

> [!TIP]  
> Your externally loaded keys should be in use now. Check by navigating to your .onion address. You can check your address the same way as described in the [setup instructions](#step-three).
