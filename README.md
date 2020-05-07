# Aerospike REST Client authentication and authorization setup

Aerospike REST Client doesn't arrive with the authentication and authorization capabilities out of the box.
This means, theoretically, anyone can access anything.

To provide an application security layer, we'll need to put the client behind the proxy server and have an OAuth2 provider around.

## Technology stack
* [Traefik](https://github.com/containous/traefik) (pronounced traffic) is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components (Docker, Swarm mode, Kubernetes, Marathon, Consul, Etcd, Rancher, Amazon ECS, ...) and configures itself automatically and dynamically. Pointing Traefik at your orchestrator should be the only configuration step you need.
* [auth-server](https://github.com/reugn/auth-server) is a simple authentication and authorization server with OAuth2 support and an Aerospike repository as a backend available.
* [Aerospike REST Client](https://github.com/aerospike/aerospike-client-rest) you already know what is this.

## Architecture diagram
![](images/auth_architecture_1.png)

## Setup
Next steps will guide you how to setup services on Docker using `docker-compose`. Other installations options available as well.

1. For docker installation please follow the guide [here](https://www.docker.com/get-started).
2. Install `docker-compose` (I'm sure you can google how-to).
3. Make sure environment variables in [docker-compose.yaml](./docker-compose.yaml) match your infrastructure confguration.
4. Run `docker-compose up -d`

Now you have proxy and auth servers up and running.

## Getting started
To start using REST client via the proxy gateway you'll need the following things:

* Acquire username and password for JWT issuing (should be set at specific namespace/set/key in Aerospike database).

* Get OAuth2 bearer token via `/token` endpoint using [basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication).
The response should look like this:
```json
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODg3MTczODUsImlhdCI6MTU4ODcxMzc4NSwidXNlciI6ImFkbWluIiwicm9sZSI6MX0.T7gCJGDQy4-efqx4rA91XdYBkswgyY2H88FzxY2I6jMFsMElfu111EY9GxyyreizyiCMe8ujLbqXHpEKMJBHzR8D7JJvMkRp1HmaSMxvzpdEIsmQTvnDJu056SwSEibCDkNAcwp70EdyrFcUbeMBEibjvFq1nGYVnRogVyzvNyAzkH0PvAnJN97yBVyotQlZdyjLhEWPRTG7CQLqx5L5UPo8phFy6gr0G7v3VAmtLxmusBaqv2AvbidVImP_nuJjvu2IBrrDIOiEnqX3e1dWe0Yg-nfFVZ7KLnYCpa6b6PJCGMsIqdRTYgckSh5fCQhIqgdZsaPBxFEl6B-o6bBUEg","token_type":"Bearer","expires_in":3600000}
```

* Send authenticated HTTP request to a REST client using `access_token` from the previous response as a bearer token within Authorization header like this:
```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODg3MTczODUsImlhdCI6MTU4ODcxMzc4NSwidXNlciI6ImFkbWluIiwicm9sZSI6MX0.T7gCJGDQy4-efqx4rA91XdYBkswgyY2H88FzxY2I6jMFsMElfu111EY9GxyyreizyiCMe8ujLbqXHpEKMJBHzR8D7JJvMkRp1HmaSMxvzpdEIsmQTvnDJu056SwSEibCDkNAcwp70EdyrFcUbeMBEibjvFq1nGYVnRogVyzvNyAzkH0PvAnJN97yBVyotQlZdyjLhEWPRTG7CQLqx5L5UPo8phFy6gr0G7v3VAmtLxmusBaqv2AvbidVImP_nuJjvu2IBrrDIOiEnqX3e1dWe0Yg-nfFVZ7KLnYCpa6b6PJCGMsIqdRTYgckSh5fCQhIqgdZsaPBxFEl6B-o6bBUEg
```
