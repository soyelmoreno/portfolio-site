# Full NGINX Tutorial - Demo Project with Node.js, Docker

[YouTube video by TechWorld with
Nana](https://www.youtube.com/watch?v=q8OleYuqntY)

These are notes from the video tutorial, specifically re: configuring Nginx. The
[Nginx configuration file](nginx.conf) in this directory is the final result of
the tutorial.

Here we are setting up Nginx to act as a load balancer. We have a simple
Node/Express application, and we have Dockerized it, then duplicated the image
into three identical containers. We want Nginx to send requests to each of the
three instances.

## Contexts

A few top-level directives, referred to as _contexts_ or _blocks_, group
together the directives that apply to different traffic types:

- `events`: General connection processing
- `http`: HTTP traffic
- `mail`: Mail traffic
- `stream`: TCP and UDP traffic

Directives placed outside of these contexts are said to be in the _main_
context.

## main context

### `worker_processes`

Controls how many parallel processes Nginx spawns to handle client requests.

What is a worker process?

Instead of using a new process for every incoming connection, Nginx uses _worker
processes_ that handle many connections using a single threaded event loop.

Worker processes are the ones that actually do the work of getting and
processing the requests coming from the browser, clients, etc. What does the
number represent?

The value is the number of worker processes Nginx should create.

Each worker process runs independently and can handle its own set of
connections.

This configuration directly influences how well it can handle traffic
(performance).

Should be tuned according to the server's hardware (CPU cores) and expected
traffic load. The more worker processes we have, the more requests Nginx can
handle.

In production, it's generally recommended to set worker processes value to be
equal to the number of CPU cores that are available on the server where Nginx is
running. If you have 4 cores, make 4 worker processes, to take advantage of
parallel processing to handle requests much faster.

For this demo, just one.

    worker_processes 1;

Could also set it to `auto` = Nginx automatically detects the number of CPU
cores available on the server and starts a corresponding number of worker nodes.

    worker_processes auto;

## `events` context (block)

Meta configurations, how to start, how performant should it be, etc.

### `worker_connections`

Per worker process: how many simultaneous connections can be opened. Let's say
it's 1024.

- If you have 1 worker process, you will be able to serve 1024 clients
- If you have 2 worker processes, you will be able to serve 2 x 1024 = 2048
  clients

The higher the value, the more requests can be handled. What to consider:

- Increases memory usage (because you have multiple connections open at the same
  time)
- The actual simultaneous connections cannot exceed the current limit on the max
  number of open files

```
events {
    worker_connections 1024;
}
```

## `http` context (block)

The main server configuration; the actual logic for how the client traffic will
be handled.

`http` = configuration specific to HTTP and affecting all virtual servers

```
http {

}
```

### MIME types

This is an optional configuration directive under the `http` block.

We can to add file types for the response to the client. We tell Nginx to
include the corresponding MIME types in the "content-type" response header, when
sending a file.

This helps the client understand how to process or render the file.

Directives:

- **include**: Tells Nginx what to include
  - **mime.types**: Please include the MIME types, i.e., the types of files,
    that you are sending back in the response to the client. Note: `mime.types`
    is actually a file that Nginx uses to map file extension to MIME types.

```
http {
  include mime.types;
}
```

### `upstream` context (block)

Upstream block defines a group of backend servers that will handle requests
forwarded by Nginx.

"upstream/downstream" are based on the flow of data:

- "Upstream servers" = Refers to traffic going _from a client towards the
  source_ or higher-level infrastructure, in this case, the application server
- "Downstream servers" = Traffic going back _from the server toward the client_
- List all the backends (i.e., all the servers) that Nginx should forward to
  (i.e., use as "upstream")

```
http {
  upstream nodejs_cluster {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
  }
}
```

### Choosing a load-balancing method

This is an optional configuration directive under the `upstream` block.

Handles the logic for how Nginx decides which server to forward the request to.
This is decided by a load-balancing algorithm.

[NGINX Doc for HTTP Load
Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

Here are some possible algrorithms:

- **Round Robin**: Default, so there is no directive. Requests are distributed
  _evenly_ across the servers.
  - Best for: Even distribution when all servers are similar.
  - Considerations: Simple, default method, doesn't account for server load or
    speed
- **Weighted Round Robin**: No directive. Requests are distributed across the
  servers, _with server weights taken into consideration_.
  - Best for: Servers with different capacities
  - Considerations: Distributes more traffic to more powerful servers
- **Least Connections**: `least_conn`: request is sent to the server with the
  least number of active connections
  - Best for: Balancing load based on connection counts
  - Considerations: Ideal when servers handle requests at different speeds
- **Weighted Least Connections**: `least_conn`: request is sent to the server
  with the least number of active connections, but also takes server weights
  into consideration
  - Best for: Unequal server capacities, dynamic requests
  - Considerations: Considers both load and server strength
- **IP Hash**: `ip_hash`: Determine the server from the client IP address
  - Best for: Session persistence (sticky sessions)
  - Considerations: Keeps clients on the same server, but fails with server
    outages
- **Generic Hash**: `hash`: Determine the server from a user-defined key
- **Least Time**: `least_time`: Determine the server with the lowest average
  latency

By default, it will use the "Round Robin" algorithm. But we can override it if
we want to.

Let's say we know that server 1 has more resources, and server 3 has the fewest
resources. Choose "Weighted Round Robin" by assigning server weights to each
server.

```
upstream nodejs_cluster {
  server 127.0.0.1:3001 weight=5;
  server 127.0.0.1:3002 weight=3;
  server 127.0.0.1:3003 weight=1;
}
```

Or let's say we want the server that is the least busy to handle the next
connection. Choose "Least Connections"

```
upstream nodejs_cluster {
  least_conn;
  server 127.0.0.1:3001;
  server 127.0.0.1:3002;
  server 127.0.0.1:3003;
}
```

### `server` context (block)

Defines how Nginx should handle requests for a particular domain or IP address

- Where/how to listen for connections
- Which domain or subdomain the configuration applies to
- How to route the requests

Directives:

- **listen**: The IP address and port on which the server will accept requests.
  Port 8080 is a standard port for local development and testing.
- **server_name**: Which domain or IP address this server block should respond
  to. The actual name that the client is sending the requests to, e.g.,
  facebook.com, example.com, etc.
- **location**: The path that the client requests, and a block with directives
  for how to handle that request
  - How specific type of requests (such as URLs or file types) should be
    handled
  - The root path (/) will apply to all requests unless more specific location
    blocks are defined

```
http {
  upstream nodejs_cluster {

  }

  server {
    listen 8080;
    server_name localhost;

    location / {

    }
  }
}
```

### `location` context (block)

We're setting up Nginx as a proxy server for multiple web servers, i.e., a
"reverse proxy". So when a request comes in to a particular path, we have to
tell Nginx what to do with the request, i.e., which server to route the request
to.

Note: When Nginx acts as a reverse proxy, the requests coming to the backend
servers originate from Nginx, _not directly from the client_.

As a result, backend servers would see the IP address of the Nginx server as the
source of the request.

```
Client IP 127.10.40.1 --->
    Nginx IP 55.60.1.1 --->
        backend receives 55.60.1.1 ❌ BAD
Client IP 127.10.40.1 --->
    Nginx IP 55.60.1.1 --->
        backend receives 127.10.40.1 ✅ GOOD
```

Therefore, our proxy needs to _forward the original request_ to the backend
servers so they can do logic on the request.

- The URL or path: /products/123
- Original IP address
- Original Host
- Referrer Information
- Custom headers

Directives:

- **proxy_pass**: Tells Nginx to "pass" the request to another server, making it
  act as a reverse proxy. The value is the upstream application server.
- **proxy_set_header**: Sets a particular header when forwarding the HTTP
  request to the upstream servers
  - Host: the original host that was requested by the client
  - X-Real-IP: the original or real IP address of the client

```
location / {
  proxy_pass http://nodejs_cluster;

  # Pass client information to the backend
  proxy_set_header Host $host
  proxy_set_header X-Real-IP $remote_addr;
}
```

## Generate SSL certificate and use it

**OpenSSL**: an open-source tool to generate keys, certificates, and managing
secure connections.

```bash
$ mkdir /Users/carlosmoreno/nginx-certs
$ cd nginx-certs
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
$ ls
nginx-selfsigned.key nginx-selfsigned.crt
```

Breakdown of the command

- `openssl req`: initiates a certificate request generation process
- `-x509`: Tells OpenSSL to output a certificate in this standard certificate
  format
- `-nodes`: Tells OpenSSL not to encrypt the private key with a passphrase
- `-days 365`: Specify validity period of the certificate, in this case 1 year
- `-keyout nginx-selfsigned.key`: Specifies the output file for the generated
  PRIVATE key, in this case "nginx-selfsigned.key"
  - Private key must be kept secret. It is never shared publicly.
  - Used to decrypt data
- `-out nginx-selfsigned.crt`: Specifies the output file for the self-signed
  certificate, in this case "nginx-selfsigned.crt". This file contains the
  PUBLIC key.
  - Public key is shared with clients to encrypt data

Information to be incorporated into the keys:

```
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:WA
Locality Name (eg, city) []:Seattle
Organization Name (eg, company) [Internet Widgits Pty Ltd]:cmoreno.me
Organizational Unit Name (eg, section) []:Engineering
Common Name (e.g. server FQDN or YOUR name) []:Carlos Moreno
Email Address []:
```

Update the server block in the Nginx config to use SSL.

```
server {
  listen 443 ssl;
  server_name localhost;
  ssl_certificate /Users/carlosmoreno/nginx-certs/nginx-selfsigned.crt;
  ssl_certificate_key /Users/carlosmoreno/nginx-certs/nginx-selfsigned.key;

  location {}
}
```

Then reload Nginx.

    nginx -s reload

Now we can naviate to https://localhost:443

Browser will load the certificate, but it will warn us that it is not signed. We
can click Advanced and proceed to localhost (unsafe) and see our site, served by
Nginx.

## Redirect HTTP ---> HTTPS

For convenience let's set up this redirect. Add a second server block that
listens to port 8080.

```
server {
  listen 8080;
  server_name localhost;

  location / {
    return 301 https://$host$request_uri;
  }
}
```

## Additional links

Tutorial on DigitalOcean: [Understanding the Nginx Configuration File Structure
and Configuration
Contexts](https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts)

Tutorial on Plesk: [NGINX Configuration
Guide](https://www.plesk.com/blog/various/nginx-configuration-guide/)

Some Nginx doc: [Configuring NGINX and NGINX Plus as a Web
Server](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/)

All Nginx directives: [Alphabetical index of
directives](https://nginx.org/en/docs/dirindex.html)

Pretty solid intro video for Nginx: [ NGINX Tutorial for Beginners: Set Up Your
Web Server in No Time!](https://www.youtube.com/watch?v=91xOfV0rdT4)
