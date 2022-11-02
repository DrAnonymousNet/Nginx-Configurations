# Introduction

Low latency, uptime, and good performance are required in today's world of the internet. During times of high traffic, the overall performance of most web applications drops, the latency rises, and sometimes the request even timeout. This often happens when the server computes power is not enough to process the workload during this period of high traffic.

The image below shows a system that could suffer from this problem:
![singleserver](https://user-images.githubusercontent.com/64500446/199344077-c4f03ded-9af4-4465-a931-754b06f88592.png)

The system makes use of a single web server to process all web requests from the client. This single server could be overworked when it receives multiple concurrent requests beyond what it could process.
 
HTTP Load Balancing is a method that could be used to mitigate this. HTTP Load balancing is a method where requests or workloads are distributed across multiple instances of a web server with the same or varying capacity profile to ensure that no single server is overworked. This optimises resource utilisation, ensures fault tolerance, and improves the overall performance of the system.
![loadbalancerarchi](https://user-images.githubusercontent.com/64500446/199344330-431c741a-021c-4801-bcf5-e7d3bdbdecd2.png)


The load balancer acts as a proxy server that accepts the request from the client. This request is then distributed across the multiple servers in a fashion that is specified by the load balancing algorithm that is configured on the load balancer. 


# Objectives

In this tutorial, we will talk about:
- NGINX as a load balancer
- The different load balancing algorithms and how to configure them.
- Pros and Cons of load balancing and possible solutions to some of these cons


# NGINX as a Load Balancer

NGINX is software that could be used as a web server, reverse proxy, http caching, mail proxy, load balancer, etc. It has been adopted by several of the busiest websites — like Adobe, and WordPress — for fast request processing and response delivery.

It is popularly used as a load balancer for high-traffic websites. If properly configured, it can serve [more than 10 thousand concurrent requests](https://en.wikipedia.org/wiki/C10k_problem) with low memory usage.

The behaviour of NGINX depends on the *context* and *directives* specified in the Nginx configuration file. Depending on the mode of installation, this configuration file can be in any of the following directories:
- `/etc/nginx/nginx.conf`
- `/usr/local/nginx/conf/nginx.conf`
- `/usr/local/etc/nginx/nginx.conf`

## Contexts, Directives and Blocks

The Nginx configuration file has a tree-like structure defined by a set of commands or statements and braces ( `{` `}` ). The statements are called *directives* which could be *block directives* or *simple directives*.

The simple directives have a name, space separated parameters terminated by a semicolon:

```
directive_name parameter_1 parameter_n ;
```
The block directives end with a brace `{ }` instead of a semicolon and they can contain inner directives.

```
directive_name parameter_n{
      inner_directive parameters;
}
```

These block directives with braces are called `context`

The inner directives are valid only within the context where they are designed for.

In the discourse of Nginx as a load balancer, the following contexts are relatively important.


![nginxcontexts](https://user-images.githubusercontent.com/64500446/199345543-e25963f9-074e-44a9-8662-fe7737520366.png)

## The Main Context

The main context is a global context that contains the directives that affect the whole application. The directives defined in this context include:
- The number of worker processes
- The location of the Log file
- The process ID etc

Unlike other contexts, the main context doesn’t define an explicit block with the use of braces. All directives with a global scope are regarded to be in the main context.

## The Events Context

Nginx uses an asynchronous event-driven approach, rather than threads, to handle requests – [wikipedia](https://en.wikipedia.org/wiki/Nginx#cite_note-Welcome-21)
The events context contains the directives that define how Nginx processes requests.
Some of the directives that are specified in this context include:
- The number of connections per worker process

## The HTTP Context

This context contains inner context and directives that determine how Nginx handles HTTP and HTTPS connections.
When Nginx is configured as a load balancer, this context contains most of the directives and inner contexts that allow nginx to act as a load balancer. Some of the directives defined in this context include:
- The address of the pool of servers to balance the load across
It also contains the server and upstream inner context.

### The Server Context

This is an inner context in the HTTP context. It contains the directives for the virtual server that respond to a request. Some of the directives defined in this context include:
- The server name
- The server port to listen to
It also contains a Location inner context.

#### The Location Context

This is an inner context in the server context. It defines how nginx responds to HTTP/HTTPS requests to a particular endpoint. You can specify custom headers, URL redirection and request distribution to upstream servers etc

### The Upstream Context

This is an inner context in the HTTP context. It defines a pool of servers that can be used for load balancing.

When configured as a load balancer, Nginx accepts requests from the clients and distributes them evenly among the multiple web servers that are specified in its upstream context. The fashion in which the loads are distributed among the upstream servers depends on the load-balancing algorithms.

# Load Balancing Algorithm
The load balancing Algorithm is a logic that is configured on the load balancer that determines how the load balancer will distribute the client’s request among the upstream servers.

Generally, The load balancing algorithms can be classified into two:
- The static load balancing algorithms
- The dynamic load balancing algorithms


## Static Load Balancing Algorithm

These algorithms do not take the current state — like the number of active connections, available resources and computing power —  of the servers into consideration while distributing the requests among the servers. They distribute the request in a predefined fashion.

### Types of Static Load Balancing Algorithms

The following are the different types of static Load Balancing algorithms:
- Round Robin
- Weighted Round Robin
- IP Hash

1. Round Robin: In this algorithm, the load balancer circularly distributes the load without any priority or consideration for the processing capacity, number of active connections and available resources of the servers. This is the default load-balancing algorithm of Nginx.
![RoundRobin](https://user-images.githubusercontent.com/64500446/199347403-cb17e8c5-05b3-4ae8-8dde-359951063f9d.png)

2. Weighted Round Robin: This algorithm is similar to the Round Robin. However, the administrator can assign weight to each server based on criteria of their choosing. The loads are distributed while considering the weight assigned to each server in the pool of servers. This algorithm is suitable when the upstream servers have varying capacity profiles.
![Weighted Round Robin](https://user-images.githubusercontent.com/64500446/199347299-a5372886-ff73-4284-b740-d2bcb1a48f7a.png)

In the image above, the first server was assigned a weight of 2 while the second and third servers were assigned a weight of one. This implies that the number of requests that will be distributed to the first server will be 2 times greater than those of the other two servers.

3. IP Hash: This algorithm hashes the IP Address of the client — sending the request — with a hashing function and sends the request to one of the servers for processing. Subsequent requests from the client’s IP address are always sent to the same server.

## Dynamic Load Balancing Algorithms

Dynamic load balancing algorithms consider the state — like available resources and number of active connections — of the server before distributing the client’s request to the upstream servers. The server that will process the request is determined by the dynamic state of the servers.

### Types of Dynamic Load Balancing Algorithms

The following are the different types of Dynamic Load Balancing algorithms:
- Least Connection
- Least Time

1. Least Connection: This algorithm distributes the client's request to servers with the least active connections at a particular time. This will ensure that no one server is overworked while other servers have fewer active connections.

2. Least Time: This algorithm distributes requests to the servers based on the average response time of the servers and the number of active connections on the server. This load-balancing algorithm is only supported by Nginx plus.


# Configurations

We will :
- spin up three local servers running on ports 8000, 8001, and 8000
- configure the load balancing algorithm for different algorithms
- balance the requests to the servers with the different algorithms we will configure.
- spin up the Nginx with docker.

We will create the servers with python’s [SimpleHTTPServer](https://docs.python.org/2/library/simplehttpserver.html) builtin library

## Creating the Servers

When you launch a server with the `SimpleHTTPServer` library:
- it loads up the `index.html` file in the directory in the browser if there is an `index.html` else
- It displays the file directory of the current working directory.

To show that the load balancing works, we will create three different `index.html` with different contents indicating which of the servers is serving the request. To do this, we will create different folders for these servers with their `index.html`.

Create a directory and `cd` into it:
```console
$ mkdir Nginx_Tuts
```
Create three different folders in this directory:
```console
$ mkdir server_1, server_2, server_3
``` 
Create an `index.html` file in each of these directories and add different contents in the html files:

```
$ cd server_1
$ echo “<h1> Served with Server 1 </h1>” >> index.html
```

```
$ cd server_2
$ echo “<h1> Served with Server 2 </h1>” >> index.html
```

```
$ cd server_3
$ echo “<h1> Served with Server 3 </h1>” >> index.html
```

You should have a file structure as shown below:

```
Nginx_Tut
├── server_1
│   └── index.html
├── server_2
│   └── index.html
└── server_3
    └── index.html

```

`cd` into each of these directories and start up the server on different ports:
```
$ cd server_1
$ python -m SimpleHTTPServer 8000 
```
You will get the following output in the command line:
```
Serving HTTP on 0.0.0.0 port 8000 ...
```
Do the same for the other servers also.

We have successfully spun up multiple local servers!

## Static Load Balancing Algorithms

We will create the `nginx.conf` file for each algorithm from scratch and we will only specify the context and directive that we need in the configuration file. We will build an Nginx docker image with the configuration file we created.


### Round Robin Configuration
As discussed earlier, The default nginx load balancing algorithm is `Round Robin` and this algorithm distributes requests to the upstream servers in a circular fashion.

Define the `http` and `events` context:
```
http{

}

events {
  
}
```

Define a `server` inner context in the `http` context and specify the port that nginx will listen to: 
```
http{
   server {
       listen 8080;
   }
}

events {
  
}
```
We specified port `8080`.

Add an *upstream* context in the `http` context that specifies the list of servers that we created earlier.
Name the upstream servers as `ourservers` so that we can identify this pool of servers with that name:

```
http{
   upstream ourservers {
       server localhost:8000;
       server localhost:8001;
       server localhost:8002
   }

   server {
       listen 8080;}
   }

}

events {
  
}

```

Add a *location* context in the server block that will process all requests sent to the base route `/` and add a [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directives that resolve and distributes request sent to this base route location to the pool of upstream servers with the name of `ourservers` that we added earlier:

http{
   upstream ourservers {
       server localhost:8000;
       server localhost:8001;
       server localhost:8002
   }

   server {
       listen 8080;
       location / {
           proxy_pass http://ourservers/;
               }
   }

}

events {
  
}
 
With this configuration, all the requests sent to the base route `/` on  `localhost` port `8080` will be proxied and passed to the server groups `ourservers` where the requests will be distributed in a round-robin fashion among the servers that we specified in the upstream block.

#### Building the Nginx Docker Image

Create a `Dockerfile` in the `Nginx_Tuts` directory and add the following:

```
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
```
In the docker configuration above:
We pulled an [alpine nginx docker image](https://hub.docker.com/_/nginx) from the docker hub.
We replaced the configuration in the `/etc/nginx/nginx.conf` with the *nginx.conf* file we created

Build a docker image from this docker file:
```
$ docker build -t loadbalancer .
```
You should get the following output after a successful build:
![dockerroundrobincreate_1](https://user-images.githubusercontent.com/64500446/199350876-8a35ca87-7ea2-43b3-8711-f6dfbcce9bca.png)

Run the Nginx container from the docker image we just built:
```
$ docker run --net:host loadbalancer
```
The [--net=host](https://docs.docker.com/network/host/) argument makes  the container’s application available on port 80 on the host’s IP address 

You should get the following output from the command above:
![dockerrunroundrobin_2](https://user-images.githubusercontent.com/64500446/199350471-3d01cec6-f388-4995-bd70-f7e0296e4298.png)


Open your browser and send a request to `http://127.0.0.1:8080`
You should get the following output:


As we notice, from the video demonstration above, the servers were picked in a circular fashion.

<video src="https://user-images.githubusercontent.com/64500446/199350654-4d352600-6f62-4542-89c0-3a361ca74281.webm" data-canonical-src="https://user-images.githubusercontent.com/64500446/199350654-4d352600-6f62-4542-89c0-3a361ca74281.webm" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;"></video>


### Weighted Round Robin

Weights can be assigned to the different servers that are configured in the upstream block directive as discussed earlier.

To configure this, we will simply assign a weight value to each of the servers in the pool of servers we had specified earlier:

```
http{
   upstream ourservers {
       server localhost:8000 weight=4;
       server localhost:8001 weight=2;
       server localhost:8002 weight=1;
   }

   server {
       listen 8080;
       location / {
           proxy_pass http://ourservers/;
               }
   }

}

events {
  
}
```
With this configuration:
The first server will process 4 times the requests that will be processed by the third server and 2 times the requests that will be processed by the second server.
The second server will process 2 times the request that will be processed by the third server

 
We will create a new docker file and build a new nginx image with this configuration file.

In the dockerfile:
```
FROM nginx:alpine
COPY weighted-rr-nginx.conf /etc/nginx/nginx.conf
```

Build an nginx image from the dockerfile:
```
$ docker build -t wrr-loadbalancer .
```
You should get the following output:
![dockerbuildwrr]()

Run a new container from the new docker image you just created:

```console
$ docker run --net=host wrr-loadbalancer
```
You should get the following output:
![wrrloadrun](https://user-images.githubusercontent.com/64500446/199351251-2b3aed91-ef0d-4a85-8c52-5d0702ae495f.png)


Open your browser and send a request to `http://127.0.0.1:8080`:

<video src="https://user-images.githubusercontent.com/64500446/199351433-6d9ca410-75a9-4990-b522-6de1e7eb14a6.webm" data-canonical-src="https://user-images.githubusercontent.com/64500446/199351433-6d9ca410-75a9-4990-b522-6de1e7eb14a6.webm" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;"></video>

As you would notice in the video above, the requests are distributed to the servers based on the weight assigned to each server.

### IP Hash Configuration

This algorithm hashes the IP Address of the client and makes sure every request from this client is served by the same server. When this server is unavailable, the request from this client will be served by another server.

To configure this, add an `ip_hash` directive in the upstream context:

```
http{
   upstream ourservers {
       ip_hash;
       server localhost:8000;
       server localhost:8001;
       server localhost:8002;
   }

   server {
       listen 8080;
       location / {
           proxy_pass http://ourservers/;
               }
   }

}

events {
  
}
```
You can build and run a new Nginx image with this configuration.

When we navigate to the `127.0.0.1:8080` in the browser, the same server will keep serving the request from my IP Address.

This is demonstrated in the video below:

<video src="https://user-images.githubusercontent.com/64500446/199352502-44df42e6-9bec-4437-8190-caf34ed235e4.webm" data-canonical-src="https://user-images.githubusercontent.com/64500446/199352502-44df42e6-9bec-4437-8190-caf34ed235e4.webm" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">

  </video>

## Dynamic Load Balancing Algorithms

### Least Connection Configuration

In the least connection algorithm, the load balancer sends the client's request to the server with the least number of active connections.

This can be configured by specifying the `least_conn` directive in the upstream context:

```
http{
   upstream ourservers {
       least_conn;
       server localhost:8000;
       server localhost:8001;
       server localhost:8002;
   }

   server {
       listen 8080;
       location / {
           proxy_pass http://ourservers/;
               }
   }

}

events {
  
}
```
You can build and run a new Nginx image with this configuration.

The video illustration is shown below:

<video src="https://user-images.githubusercontent.com/64500446/199352585-32668481-87d5-42f8-a04c-0522c6801821.webm" data-canonical-src="https://user-images.githubusercontent.com/64500446/199352585-32668481-87d5-42f8-a04c-0522c6801821.webm" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;"></video>


## Least Time Configuration

With this configuration, Nginx distributes requests to the servers based on the average response time as well as the number of active connections. In addition, different weights can be assigned to these servers depending on the capacity profile of the servers. If weight is assigned to each server, The weight parameter will be taken into consideration alongside the average response time and the number of active connections.

This algorithm can be configured by adding a `least_time` directive to the *upstream* context.

The average response time of the servers is either based on the time to receive the response header or the response body. This is controlled by the `header` and `last_byte` parameters in the `least_time` directive. There is a third optional parameter called `inflight` that indicates whether the response time of all requests will be tracked or just the response for successful requests.

The configuration below shows the *least_time* configuration that uses the average time of the response body to track the average response time. 
```
http{
   upstream ourservers {
       least_time last_byte;
       server localhost:8000;
       server localhost:8001;
       server localhost:8002;
   }

   server {
       listen 8080;
       location / {
           proxy_pass http://ourservers/;
               }
   }

}

events {
  
}

```
The video description below assumes all servers have the same number of active connections:

<video src="https://user-images.githubusercontent.com/64500446/199364470-606ed16a-0c3e-4090-bd81-2b23c6fbe716.webm" data-canonical-src="https://user-images.githubusercontent.com/64500446/199364470-606ed16a-0c3e-4090-bd81-2b23c6fbe716.webm" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;"></video>

# Pros and Cons of Load Balancing


In this section:
I will discuss some of the advantages and performance benefits of Load Balancing
I will also discuss some of the things that could go wrong and the possible solutions to them

# Conclusion
In this tutorial, we discussed:
- Nginx as a load balancer
- the static and dynamic load-balancing algorithms
- the Round Robin, Weighted Round Robin and the IP Hash load balancing algorithms from the static type
- the Least connection and Least load-balancing algorithms
- We also configured these different types of algorithms as well as showed a video illustration of them.
- Lastly, we discussed some of the pros and cons of this technique.

The configuration and files used in this tutorial can be found in this [GitHub repository](https://github.com/DrAnonymousNet/Nginx-Configurations)
