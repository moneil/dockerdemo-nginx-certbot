# Example static website with Docker, Nginx and Certbot

April 04, 2020

This is the full project source and directory layout behind the *to be written* Blackboard article [**How to dockerize a static website with Nginx, automatically renew SSL certificates for your domain with Certbot and deploying to your Docker host.**](https://docs.blackboard.com/learn/Dockerizing%20NGINX.html) 

This setup is useful for routing microservices, and LTI or REST applications using NGINX.

A huge thank you to [Vic Shóstakkoddr](https://github.com/koddr) (aka Koddr) for the basis of this project!

## Requirements

- Docker `19.03.6, build 369ce74a3c+`
- Docker-compose `1.21.2, build a133471+`

## Usage
Deploying this project is done in ten easy steps which are individually outlined below:

1. Create your Docker host system and FQDN (required)
2. GIT setup
3. Edit the filed to reflect the Docker host FQDN in the config and yaml files
4. Rename the NGINX FQDN config file
4. Docker host - Install Docker and docker-compose executables
6. Docker host - Check configuration of `Certbot` 
8. Docker host - Create a production certificate
9. Docker host - test NGINX configuration
10. Docker host - deploy NGINX

Note that while the below details are sufficient for setting up and deploying this project, you may find further 

## Step 1: Create your Docker host system and FQDN
It is beyond the scope of this how-to to delve into standing up a server and network setup. For the purposes of this project I used an Amazon AWS Ubuntu AMI and used noip.com for establishing a test domain.

Once you have your test domain up you should be able to ssh to your server using the username@sub.domain.tld e.g. `$ ssh -i <your aws .pem> ubuntu@sub.domain.tld`

## Step 2: GIT Setup
### Clone this repository
This step is optional for your local system because you may choose instead to clone to your Docker host, make edits there, and push to your github repo via a terminal session. These instructions assume working on the Docker host only.

### GitHub
SSH to your Docker host and clone this repo:
	```$ git clone https://github.com/moneil/dockerdemo-nginx-certbot.git nginx
cd nginx
```

## Step 3: Edit the host FQDN in the config and yaml files
The container and scripts need to know your system's FQDN. This information is changed in the following three files

- ./webserver/nginx/site.domain.tld
- ./webserver/nginx/default.conf
- ./docker-compose.prod.yml

In all cases replace site.domain.tld with your FQDN.

You may run `$ grep -rl "site.domain.tld" .` to discover the files on your system.


## Step 4: Rename NGINX FQDN config file
Rename the `./webserver/nginx/site.domain.tld` file to `./webserver/nginx/<FQDN>`

## Step 5: Docker host - install Docker and docker-compose executables
If `$ docker --version` and `docker-compose --version` do not return with the required versions then you must install them. 

Installation instructions are available via the [Docker website](https://docs.docker.com/install/)

## Step 6: Docker host - Check configuration of `Certbot`
To check configuration of `Certbot` and create a test certificate, start the process of obtaining SSL certificate in test mode:

```console
make certbot-test DOMAINS="site.com www.site.com" EMAIL=mail@site.com
```
If you see `Congratulations!` Let's Encrypt can reach your server and create  certificates. If not check your network/FQDN for errors (You should be able to ssh to the docker host using the FQDN. 

Note that Let's Encrypt limits certificate request attempts to 50 per week so do not continue until you have successfully tested that certbot can acceess your system and that the FQDNs are accurate in the above files.

## Step 7: Docker host - create a production certificate
Now that you know certbot can reach your Docker host via your FQDN, you can request a production Let's Encrypt certificate:

```console
make certbot-prod DOMAINS="site.com www.site.com" EMAIL=mail@site.com
```

When this process completes you may test your NGINX configuration...


## Docker host: test NGINX configuration
```console
make deploy-test
```

If you have no errors you may deploy the container to production.

## Docker host: deploy NGINX

No errors? Go your static website to production:

```console
make deploy-prod
```

Here is where you can use `$ docker logs` to see the results or troubleshoot.

1. `$ docker container ls` will return a list of containers. 
2. Note the CONTAINER ID for the nginx:alpine IMAGE (e.g.: bfc6b902a4e6) and request the logs:
`$ docker logs bfc6b902a4e6`

## Success!
You have deployed an SSL secured NGINX container which will auto-refresh the certificate following Let's Encrypt best practices.

You may now edit the file in `./webserver/nginx/<FQDN>` to route requests to your services or applications by adding location proxies.


## Author

- [Mark O'Neil](https://github.com/moneil).

## Article assistance

If you want to say «thank you»:

1. Twit about article [on your Twitter](https://twitter.com/intent/tweet?text=How%20to%20dockerize%20your%20static%20website%20with%20Nginx%2C%20automatic%20renew%20SSL%20for%20domain%20by%20Certbot%20and%20deploy%20it%20to%20DigitalOcean%3F%20https%3A%2F%2Ftwitter.com%2Fintent%2Ftweet%3Ftext%3Dhttps%3A%2F%2Fdev.to%2Fkoddr%2Fhow-to-dockerize-your-static-website-with-nginx-automatic-renew-ssl-for-domain-by-certbot-and-deploy-it-to-digitalocean-4cjc).
2. Add a GitHub Star and make Fork to this repository.

Thanks for your support! 😘

## License

MIT
