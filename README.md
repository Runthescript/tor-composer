# TOR Service Composer

Use this compose setup to generate a Tor hidden service with Nginx serving static assets. Simply replace the contents of the folder `your-project` with your static assets. Your `index.html` file should be located in `tor-composer/your-project/index.html`.

## Setup Instructions

Once you have added all your files, in the terminal, at the root of your project, issue the following command:

```bash
docker-compose up --build
```

Once you see that the build is complete and both containers are running, you can enter the container with the following command:

```bash
docker exec -it tor /bin/sh
```

Inside the tor container, issue the following command to print your .onion address:

```bash
cat /var/lib/tor/hidden-services/hostname
```

This will print your .onion address to the terminal. Open the Tor Browser and navigate to your newly hosted site.
Backup Your Data

It would be a good idea to backup your Docker volume to avoid losing your hidden-service keys and address.

### Adding an external hidden-service file

If you already have an .onion address and would like to use it, you may load your keys into the container using the following method. Before you start the container, move the folder containing the keys, address, and authorized_clients into the projects hiddenservices directory. If your folder that your are moving to the project is not currently named hidden-services, rename it before you run the docker compose command. In the docker-compose.yml file you will need to uncomment the line below the comment:

```bash
    volumes:
      - ./torrc.template:/etc/tor/torrc.template
      - torkeys:/var/lib/tor/
      - ngxaccesslog:/var/log/tor/host.access.log
      # Uncomment the line below to add key files directory
      # - ./hiddenservices:/usr/tor/home/
    restart: always
```

uncomment like this:

```bash
    volumes:
      - ./torrc.template:/etc/tor/torrc.template
      - torkeys:/var/lib/tor/
      - ngxaccesslog:/var/log/tor/host.access.log
      # Uncomment the line below to add key files directory
      - ./hiddenservices:/usr/tor/home/
    restart: always
```

We will need to enter the container as root and assign some permissons to the folder so that we can copy our keys into the HiddenServiceDir. In the terminal navigate to the root of the tor-composer project. You will need to start the containers with

```bash
docker-compose up --build
```

Once the container has started type or paste the following command to enter the container.

```bash
docker exec -u root -it tor /bin/sh
```

Now that you are in the container check to make sure your key files are copied and what user owns them with the following

```bash
ls -l /usr/tor/home/hidden-services
```

which should give you an output that looks something like this

```bash
-rwx------ 1 root root        hostname
-rwx------ 1 root root        hs_ed25519_public_key
-rwx------ 1 root root        hs_ed25519_secret_key
```

Once you know your files are in the container you can use the following commands to fix permissions and copy the data

```bash
chown -R tor:tor /usr/tor/home/hidden-services
chmod -R 700 /usr/tor/home/hidden-services
cp /usr/tor/home/hidden-services/* /var/lib/tor/hidden-services
cat /var/lib/tor/hidden-services/hostname
```

If after running the last command you see your onion address printed to the terminal you can now exit the container with ctrl+d or by typing exit and hitting enter. After exiting container you will

```bash
docker-compose down
```

Go back to the docker-compose.yml file and recomment the line we uncommented. It should look like the example below when done.

```bash
    volumes:
      - ./torrc.template:/etc/tor/torrc.template
      - torkeys:/var/lib/tor/
      - ngxaccesslog:/var/log/tor/host.access.log
      # Uncomment the line below to add key files directory
      # - ./hiddenservices:/usr/tor/home/
    restart: always
```

Finally you can go ahead in the root of tor-composer project

```bash
docker-compose up --build
```

Your externally loaded keys should be in use now. Check by navigating to your .onion address. You can check your address the same way as described in the setup instructions.

```

```
