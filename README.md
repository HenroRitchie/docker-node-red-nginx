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

## Copyright

## Details
