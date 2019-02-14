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
Docker is used to provide multiple containers to run the different components of this setup. Docker is therefore a pre-requist, as well as Docker Compose which is used to configure the multiple containers as well as orchestrate them. The installation has been tested on Digital Ocean using a pre-built Docker droplet. Docker-Compose were installed manually after droplet setup and configuration.

For more details please consult the following websites for more information:
[Docker](https://www.docker.com/)
[Docker-Compose](https://docs.docker.com/compose/)
[Digital Ocean Docker Droplet](https://www.digitalocean.com/products/one-click-apps/docker/)

### Node-Red
Node-Red is flow based programming tool with an infinite number of applications. Node-Red Dashboard provides a set of nodes to easily create dashboards and user interfaces. However, the Node-Red dashboard is inherently a single user setup. The only why to provide multiple user capabilities is to run numerous Node-Red instances. Each instance has its own dashboard. In addition to the standard Node-Red Dashboard the setup also includes Node-Red Worldmap to provide each instace the capability to plot items on a map.

### NGINX
WIP
