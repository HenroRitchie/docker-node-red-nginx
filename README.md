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
* PostgreSQL - SQL database used to store information (or any other data storage)
* PGAdmin - Used to manage PostGresSQL

## Disclaimer
I am not specialist in software, nor in any of the different components used. I cannot warrant this setup will work or will be free of faults and/or other complications.

## Copyright
All the copyright for the different components used resides with the creators of the respective components.

## Details
### Docker
Docker is used to provide multiple containers to run the different components of this setup. Docker is therefore a prerequist, as well as Docker Compose which is used to configure the multiple containers as well as orchestrate them. The installation has been tested on Digital Ocean using a pre-built Docker droplet. Docker-Compose were installed manually after droplet setup and configuration.

For more details please consult the following websites for more information:
* [Docker](https://www.docker.com/)
* [Docker-Compose](https://docs.docker.com/compose/)
* [Digital Ocean Docker Droplet](https://www.digitalocean.com/products/one-click-apps/docker/)

#### Below the Docker Compose explanation:
The first part starts NGINX as reverse proxy. The details of the NGINX implementation follows below. The NGINX configuration is stored outside the instance for ease of manipulation. In future the certificates for SSL implementation will also be stored outside the container. This allows a separate process to request new certificates before the instance is restarted to make the new configuration or certificates valid.
```
  proxy:
    image: nginx:latest
    restart: always
    volumes:
      - /home/nginx_data/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - postgres
      - nodered0
      - nodered1
    networks:
      - net
    ports:
      - "80:80"
      - "443:443"
```
The second part starts all the Node-Red instances. Each Node-Red instance has its own storage and the volumes are defined at the end of the Docker Compose file. The current Docker Compose file uses the standard Node-Red instance from Docker Hub, however, this will be changed to use a custom image. The custom image will include a couple of additional nodes, especially the Dashboard nodes, the Worldmap node and will be configured to use projects from the start. Github may be included to store the Node-Red projects. Node-Red instances with a single numerical denominator is used for system flows; i.e. the login page and authentication route. Node-Red instances with a numerical flow of 6 numerals are used for users.
```
  nodered0:
    image: nodered/node-red-docker:latest
    volumes:
     - nodered0_data:/data
    restart: always
    networks:
     - net

  nodered1:
    image: nodered/node-red-docker:latest
    volumes:
     - nodered1_data:/data
    restart: always
    networks:
     - net

  nodered100001:
    image: nodered/node-red-docker:latest
    volumes:
     - nodered100001_data:/data
    restart: always
    networks:
     - net

  nodered100002:
    image: nodered/node-red-docker:latest
    volumes:
     - nodered100002_data:/data
    restart: always
    networks:
     - net
```
The third part defines addtional containers used in the system. This may include various other software packages and databases. In this example PostgreSQL is included and PGAdmin to administrate PostgreSQL.

```
  postgres:
    image: postgres:11
    restart: always
    volumes:
      - /home/postgres_data:/var/lib/postgresql/data
    networks:
      - net
    environment:
      - POSTGRES_DB=users
      - POSTGRES_USER=dbuser
      - POSTGRES_PASSWORD=*****

  pgadmin:
    image: dpage/pgadmin4
    restart: always
    volumes:
     - pgadmin_data:/var/lib/pgadmin
    networks:
      - net
    ports:
      - "8080:80"
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@foo.bar
      - PGADMIN_DEFAULT_PASSWORD=*****
```
The fourth part defines the internal network used between the docker containers to communicate with one another. This is especially important for the communication between the reverse proxy and the containers. 

```
volumes:
  nodered0_data:
  nodered1_data:
  nodered100001_data:
  nodered100002_data:
  nodered100003_data:
  nodered100004_data:
  nodered100005_data:
  nodered100006_data:
  nodered100007_data:
  nodered100008_data:
  nodered100009_data:
  nodered100010_data:
  pgadmin_data:

networks:
  net:
```

##### To Do
* Certificates must be added for the SSL implementation.
* At the moment the NGINX configuration file stores all the information for all the instances. This should be broken into separate files and stored in the 'Sites Available' and 'Site Enabled' folders. This would decrease the complexity involved in maintaining the NGINX implementation.
* Create custom Node-Red image with the Dashboard nodes and the Worldmap node included.
* Configure the Node-Red image to use projects by default.

### Node-Red
Node-Red is flow based programming tool with an infinite number of applications. Node-Red Dashboard provides a set of nodes to easily create dashboards and user interfaces. However, the Node-Red dashboard is inherently a single user setup. The only way to provide multiple user capabilities is to run numerous Node-Red instances. Each instance has its own dashboard. In addition to the standard Node-Red Dashboard the setup also includes Node-Red Worldmap to provide each instance the capability to plot items on a map.

Each Node-Red container provides a single Node-Red instance. In the rest of this readme Node-Red instances refers to a Node-Red containers.

### NGINX
NGINX has two main purposes. 

The first purpose is to act as a reverse proxy. The reverse proxy provides a well formatted URI link to the different Node-Red instances. As each of the Node-Red instances terminates on the standard Node-Red port (1880) and each of the instances are connected via the internal Docker name the reverse proxy provides configurable routes to these instances. The reverse proxy also allows the routes to be 'grouped' together by defining the path to the instance. As mentioned earlier the NGINX configuration will be updated. Below the snippet detailing a typical reverse proxy setup. The user instance ID is 100001 in this case.
```
    location /100001/ui/ {
        proxy_pass http://nodered100001:1880/ui/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        auth_request /auth;
    }

    location /100001/worldmap/ {
        proxy_pass http://nodered100001:1880/worldmap/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        auth_request /auth;
    }

    location /admin/100001/ {
        proxy_pass http://nodered100001:1880/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        auth_request /auth;
    }
```
Note the difference between the route to the UI and the Worldmap, and the route to the standard Node-Red Admin interface. This difference will be explained shortly.

The second purpose is to control access to the instances. Every time a specific route is accessed, as per the reverse proxy implementation, the access is authenticated. NGINX does this by passing every incoming message to the authentication route. The authentication route checks the credentials and allows/denies access.

The current implementation works as explained below. It does have some issues and security concerns which will be adressed in future:
1. When a user attempt to access their instance the request is sent to the authentication route. The authentication route checks the credentials and replies with a HTTP 200 response if access is allowed or a HTTP 401 response of if denied. If it is the first time the user requests access it will be denied and the user will be redirected to the login page.
1. On the login page the user is required to enter their username and password. The username is used to retrieve the instance ID which belongs to the user. The username, password, instance ID and a time stamp is then compiled into a single message. This message is then encrypted using BCRYPT and loaded into a cookie. The cookie is sent back to the user's browser and the user is redirected to their instance. The cookie contains three items.
   1. The value field, which contains the username, password, Instance ID and time stamp.
   1. The max age, which defines the expiry date of the cookie.
   1. The path, which defines where the cookie is valid. 
      * The path of the user instance id is defined as '/100001/ui' and '/100001/worldmap'
      * The cookie is set to access the '/100001/' path. This allows the cookie to be valid for both the ui and the worldmap URI's.
      * For the admin interface the reverse proxy defines the require path as '/admin/'. This allows a admin user, which usaully has the most power, to access all the different Node-Red admin interfaces with a single login and cookie. All the admin interfaces are available on '/admin/100001/', '/admin/100002/' etc.
1. The reverse proxy receives the request from the user's browser to access their instance. The previously generated cookie is sent with. The reverse proxy request the authentication route to verify the login information before allowing access. The cookie is decrypted to obtain the username and password of the user. The username is used to retrieve the password from the database. The password is first decrypted as it is stored encrypted in the database, and then compared to the provided password. If the two passwords match the authentication route responds with an HTTP 200 result. If anything in this process fails or if the password is incorrect the authetication route responds with an HTTP 401 result.
1. If a HTTP 200 result is received the reverse proxy redirects the user as per the path defined in the cookie. This basically allows the user access to their dashboard.
1. If a HTTP 401 result is received the user is directed back to the login screen.
```
    #-----Redirect on 401 Error-----
    error_page 401 = @error401;

    # If the user is not logged in, redirect them to login URL
    location @error401 {
        return 302 /;
    }
```
The login page is provided by Node-Red instance 0. Node-Red instance 0 also provides the login screen for the admin interface. In the code snippet below Node-Red 0 configuration has been changed slightly for the admin interface to be available on the '/admin/' URI and the static login page to be on '/'. This will be change back to the standard setup because the the reverse proxy can do all the routing and changing the standard Node-Red config adds additional work.
```
    #NR instance for login page
    location / {
        proxy_pass http://nodered0:1880/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    #Admin interface of login page
    location /admin/0/ {
        proxy_pass http://nodered0:1880/admin;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        auth_request /auth;
    }

    #Authentication route
    location /auth {
        proxy_pass http://nodered1:1880/auth/;
        proxy_pass_request_body off; # no need to send the POST body

        proxy_http_version 1.1;
        proxy_set_header Content-Length "";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    #Admin interface of authentication route
    location /admin/1/ {
        proxy_pass http://nodered1:1880/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        auth_request /auth;
    }
```

The authentication route is provided by Node-Red instance 1. This instance will be exteremly busy when many users connect to their instances. In this setup the authentication route queries the database everytime it receives a authentication request. This is inefficient and the should probably be changed to load the complete user database into memory if possible. This has not been stress tested at all.

##### To Do
* Change the authentication route to load the complete user database into memory.
* Investigate JWT to provide access in place of storing a cookie with the both the username and password, which is less secure.

## Project To Do
* Include all the required Node-Red flows to achieve a working setup
* Provide complete Docker Compose, and NGINX config files
