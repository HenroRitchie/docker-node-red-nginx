# docker-node-red-nginx
A Node-Red multi-instance/multi-user setup built with Docker and NGINX as reverse proxy
## Synopsis
A requirement exist for a setup which is able to provide multi-user capability to Node-Red, specifically the Node-Red dashboard. As stated on the Node-Red Dashboard page:

> Multiple Users
> - This Dashboard does NOT support multiple individual users. It is a view of the status of the underlying Node-RED flow, which itself is single user. If the state of the flow changes then all clients will get notified of that change.

The following setup is an attempt to address this. It uses a number of different components to fulfill all the requirements. 

* Docker - to provide multiple containers to run all of the other components
* Node-Red - The components multi-user capability is required for, and in future to manage the system
* Nginx - to provide a reverse proxy to control access and routing
* PostGresSQL - SQL database used to store information (or any other data storage)
* PGAdmin - Used to manage PostGresSQL

## Disclaimer
I am not specialist in software, nor in any of the different components used. I cannot warrant this setup will work or will be free of faults and/or other complications.

## Copyright
All the copyright for the different components used resides with the creators of the respective components.

## Details
### Docker
Docker is used to provide multiple containers to run the different components of this setup. Docker is therefore a prerequist, as well as Docker Compose which is used to configure the multiple containers as well as orchestrate them. The installation has been tested on Digital Ocean using a pre-built Docker droplet. Docker-Compose were installed manually after droplet setup and configuration.

For more details please consult the following websites for more information:
[Docker](https://www.docker.com/)
[Docker-Compose](https://docs.docker.com/compose/)
[Digital Ocean Docker Droplet](https://www.digitalocean.com/products/one-click-apps/docker/)

#### Below the Docker Compose explanation:
The first part starts NGINX as reverse proxy. The details of the NGINX implementation follows below. The NGINX configuration is stored outside the instance for ease of manipulation. In future the certificates for SSL implementation will also be stored outside the container. This allows a separate process to request new certificates before the instance is restarted to make the new configuration or certificates valid.
```code snippet```
To Do
	• Certificates must be added for the SSL implementation.
	• At the moment the NGINX configuration file stores all the information for all the instances. This should be broken into separate files and stored in the 'Sites Available' and 'Site Enabled' folders. This would decrease the complexity involved in maintaining the NGINX implementation.
The second part starts all the Node-Red instances. Each Node-Red instance has its own storage and the volumes are defined at the end of the Docker Compose file. The current Docker Compose file uses the standard Node-Red instance from Docker Hub, however, this will be changed to use a custom image. The custom image will include a couple of additional nodes, especially the Dashboard nodes, the Worldmap node and will be configured to use projects from the start. Github may be included to store the Node-Red projects. 
The third part defines the internal network used between the docker containers to communicate with one another. This is especially important for the communication between the reverse proxy and the containers. 

### Node-Red
Node-Red is flow based programming tool with an infinite number of applications. Node-Red Dashboard provides a set of nodes to easily create dashboards and user interfaces. However, the Node-Red dashboard is inherently a single user setup. The only way to provide multiple user capabilities is to run numerous Node-Red instances. Each instance has its own dashboard. In addition to the standard Node-Red Dashboard the setup also includes Node-Red Worldmap to provide each instance the capability to plot items on a map.

### NGINX
NGINX had two main purposes. 

The first purpose is to act as a reverse proxy. The reverse proxy provides a well formatted URI link to the different Node-Red instances. As each of the Node-Red instances terminates on the standard Node-Red port (1880) and each of the instances are connected via the internal Docker name the reverse proxy provides configurable routes to these instances. The reverse proxy also allows the routes to be 'grouped' together by defining the path to the instance. As mentioned earlier the NGINX configuration will be updated. Below the snippet detailing a typical reverse proxy setup. 

Note the difference between the route to the UI and the Worldmap, and the route to the standard Node-Red Admin interface. This difference will be explained shortly.

The second purpose is to control access to the instances. Every time a specific route is accessed, as per the reverse proxy implementation, the access is authenticated. NGINX does this by passing every incoming message to the authentication route. The authentication route checks the credentials and allows/denies access. 
The current implementation works as follow:
1. When a user attempt to access their instance the request is sent to the authentication route. The authentication route checks the credentials and replies with a HTTP 200 response if access is allowed or a HTTP 401 response of if denied. If it is the first time the user requests access it will be denied and the user will be redirected to the login page.
1. On the login page the user is required to enter their username and password. 

