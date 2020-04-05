# dockerdemo-nginx-certbot: setting up a secure NGNIX website with Docker using Certbot for auto-renewed SSL certificates

April 04, 2020

This is the full project source and directory layout for a Docker deployable SSL-secured NGINX server which uses Certbot to automantically renew SSL certificates for your domain.

This setup is useful for routing microservices, and LTI or REST applications using NGINX.

A huge thank you to [Vic Shóstakkoddr](https://github.com/koddr) (aka Koddr) for the basis of this project!

## Requirements

- Ubuntu 18.04.3 LTS (Bionic Beaver)
- Docker `19.03.6, build 369ce74a3c+`
- Docker-compose `1.21.2, build a133471+`

## Usage
Deploying this project is done in nine easy steps which are individually outlined below:

1. Create your Docker host system and FQDN (required)
2. GIT setup
3. Edit files to reflect the Docker host FQDN in the config and yaml files
4. Rename the NGINX FQDN config file
5. Docker host - Install Docker and docker-compose executables
6. Docker host - Check configuration of `Certbot` 
7. Docker host - Create a production certificate
8. Docker host - Test NGINX configuration
9. Docker host - Deploy NGINX

These steps must be done in order.

## Step 1: Create your Docker host system and FQDN
This how-to does not delve into standing up a server and network setup. For the purposes of this project I used an Amazon AWS Ubuntu AMI and used noip.com for establishing a test domain.

Once you have your test domain up, you should be able to ssh to your server using the username@sub.domain.tld e.g. `$ ssh -i <your aws .pem> ubuntu@<insert your FQDN>`

Make certain that the following ports are open on your Docker host system:

1. ssh:22
1. http:80
1. https:443

## Step 2: GIT Setup
### Clone this repository
This step is optional for your local system because you may choose instead to clone to your Docker host, make edits there, and push to your github repo via a terminal session. These instructions assume you are working on the Docker host only.

### GitHub
SSH to your Docker host and clone this repo:

```
$ git clone https://github.com/moneil/dockerdemo-nginx-certbot.git nginx
$ cd nginx
```

## Step 3: Edit the host FQDN in the config and yaml files
The container and scripts need to know your system's FQDN. This information is changed in the following three files

- ./webserver/nginx/site.domain.tld
- ./webserver/nginx/default.conf
- ./docker-compose.prod.yml

In all cases replace site.domain.tld with your FQDN.

To discover the files on your system you may run: `$ grep -rl "site.domain.tld" .` 

You may also run a search and replace: `grep -rlZ 'site.domain.tld' . | xargs -0 sed -i.bak 's/site.domain.tld/<insert your FQDN>/g'`

Test success: 

```
$ grep -rl "site.domain.tld" .
nginx/.git/index.bak
nginx/docker-compose.prod.yml.bak
nginx/README.md.bak
nginx/webserver/nginx/default.conf.bak
nginx/webserver/nginx/site.domain.tld.bak

$ grep -rl "<insert your FQDN>" .
nginx/.git/index
nginx/docker-compose.prod.yml
nginx/README.md
nginx/webserver/nginx/default.conf
nginx/webserver/nginx/site.domain.tld

```
and then clean up the .bak files: `grep -rlZ 'site.domain.tld' . | xargs -0 rm`

Test Success:
```
$ grep -rl "site.domain.tld" .
nginx/.git/index.bak
nginx/docker-compose.prod.yml.bak
nginx/README.md.bak
nginx/webserver/nginx/default.conf.bak
nginx/webserver/nginx/site.domain.tld.bak

$ grep -rl "your FQDN" .
nginx/.git/index
nginx/docker-compose.prod.yml
nginx/README.md
nginx/webserver/nginx/default.conf
nginx/webserver/nginx/site.domain.tld

```

## Step 4: Rename NGINX FQDN config file
Rename the `./webserver/nginx/site.domain.tld` file to `./webserver/nginx/<FQDN>`

## Step 5: Docker host - install Docker and docker-compose executables
If `$ docker --version` and `docker-compose --version` do not return with the required versions then you must install them. 

Installation instructions are available on the [Docker website](https://docs.docker.com/install/)

There are some post install steps required to run the scripts/docker as a non-root user:

1. Add the Docker group: `$ sudo groupadd docker`
2. Add your user to the group: `$ sudo usermod -aG docker $USER`

Log out and log back in so that your group membership is re-evaluated.

You should now be able to run docker as a non-root user:
`$ docker run hello-world`

To start docker deamon on boot:
`$ sudo systemctl enable docker`

Find the newest version of docker-compose on the release page at GitHub or by curling the API if you have jq installed - follow prompts:

`$ VERSION=$(curl --silent https://api.github.com/repos/docker/compose/releases/latest | jq .name -r)`

Download and set permissions:

```
$ DESTINATION=/usr/local/bin/docker-compose
$ sudo curl -L https://github.com/docker/compose/releases/download/${VERSION}/docker-compose-$(uname -s)-$(uname -m) -o $DESTINATION
$ sudo chmod 755 $DESTINATION
```

## Step 6: Docker host - Check configuration of `Certbot`
To check the configuration of `Certbot` and create a test certificate, start the process of obtaining SSL certificate in test mode:

```
$ make certbot-test DOMAINS="your FQDN" EMAIL=your@email
```
If you see `Congratulations!` Let's Encrypt can reach your server and create  certificates. If not check your network/FQDN for errors (You should be able to ssh to the docker host using the FQDN. 

Note that Let's Encrypt limits certificate request attempts to 50 per week. Do not continue until you have successfully tested that certbot can access your system and that the FQDNs are accurate in the above files.



## Step 7: Docker host - create a production certificate
Now that you know certbot can reach your Docker host via your FQDN, you can request a production Let's Encrypt certificate:

```
$ make certbot-prod DOMAINS="your FQDN" EMAIL=your@email
```
When prompted select: 2) Remove registered domains and continue

If you see `Congratulations!` Let's Encrypt can reach your server and create  certificates. If not check your network/FQDN for errors (You should be able to ssh to the docker host using the FQDN. 

When this process completes you may test your NGINX configuration...


## Step 8: Docker host: test NGINX configuration
```
$ make deploy-test
```

If you have no errors you may ctrl-c to exit the script.

## Step 9: Docker host: deploy NGINX

You may now start your static website in production:

```
$ make deploy-prod
```

Here is where you can use `$ docker logs` to see the results or troubleshoot.

1. `$ docker container ls` will return a list of containers. 
2. Note the CONTAINER ID for the nginx:alpine IMAGE (e.g.: bfc6b902a4e6) and request the logs:
`$ docker logs bfc6b902a4e6`

## Success!
Congratulations! 

You have deployed an SSL secured NGINX container which will auto-refresh the certificate following Let's Encrypt best practices.

You may now edit the file in `./webserver/nginx/<FQDN>` to route requests to your services or applications by adding location proxies.


## Author

- [Mark O'Neil](https://github.com/moneil).


## License

MIT
