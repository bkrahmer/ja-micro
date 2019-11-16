# Ja-micro [![License](https://img.shields.io/:license-apache-blue.svg)](https://opensource.org/licenses/Apache-2.0) 

Ja-micro is a lightweight Java framework for building microservices.

## Introduction ##

Ja-micro is a framework that allows developers to easily develop microservices in 
Java. It was developed beginning in 2016, during a push to create a new platform 
based on Kubernetes. With version 4, the focus is on easy integration with OpenShift, 
and using standards such as OpenApi, gRPC and AMQP.  It also supports database 
migrations, Infinispan distributed memory features, and a rich Kafka integration.  It 
includes a Gradle wrapper for elimination of boilerplate and simultaneously allow rich 
customization of build pipelines. 

The framework takes care of many of the concerns needed in a modern containerized 
application so that developers can simply focus on the business functionality 
of their services rather than tedious boostrapping.  Cloud native principles 
are used wherever possible, and example code explores other best practices.

## Audience ##

The audience for Ja-micro is engineers who want to quickly build microservices for the 
cloud.  With seamless integrations for many of the top technologies and software products,
you can focus entirely on building productive business logic, not repeatedly building 
up monotonous scaffolding and the tests to prove that the integration is working.  
If you have the interface for a service in mind, you can get started writing code within
ten minutes.  There are many different use cases that Ja-micro can handle.  To minimize
start-up speed and make the executable artifact as small as possible, you choose which
ports and adapters to include in your service.

## Features ##

* Simply build service an OCI container or fat jar.
* Configuration from environment variables, the command-line or external configuration services.
* Standardized json logging.
* Standardized metrics reporting
* Simple interface for calling endpoints on other services and handling errors from them.
* Simple interface for a service to support readiness and health checks.
* Database migrations built-in.
* Simplified event-handling using Kafka or AMQP.
* Guice dependency injection for ease of implementation and testing scenarios.
* Components to create service integration (contract) testing scenarios.
* Create services that are part of an event-driven architecture

## Details

### RpcClient and Error Handling ###

In order to allow a service to call other services, there is support to easily create `RpcClient`s.
An `RpcClient` is an object that abstracts an rpc endpoint of another service.  One can create an `RpcClient`
by using the `RpcClientFactory`. Once one has an `RpcClient`, calling another service endpoint is as simple
as calling the `callSynchronous` method (a future version of the framework will also
support asynchronous calls) with a protobuf request message, and it will return a protobuf response
message. 

The timeout policy, the retry policy, and the error-handling are all handled by the client. By default,
we have said that the default retry policy should be to retry a failed request one time, if the response is
retriable. In a future release, we will add support for time budgeting, where a client of a service can set
how much time is allowed to service the whole request. For now, we have a static policy
with a default timeout of 1000ms, and this can be customized per client. 

The `RpcCallException` class defines all of our exception categories, their default 
retriable setting, and what the resulting HTTP status code is (when using
an HTTP transport). When one service calls another (client calling server), if a server throws an exception
during processing the response, the exception is transparently transported back to the client and can be
rethrown on the client-side (ignoring the retry aspect here).

Default RpcClient retry policy (1) can be overridden with setting 'rpcClientRetries'.
Default RpcClient timeout policy (1000ms) can be overridden with setting 'rpcClientTimeout'.

### Readiness and Liveness Probes ###
 
In a microservice architecture, you want your orchestrator (Kubernetes / OpenShift) to
know when a service is ready to handle requests when it first starts up.  For this, there
is support for developers to easily create readiness probe endpoints.

Similarly, sometimes services may no longer be functional enough to serve traffic.  Liveness
probes can be configured so that the orchestrator can know when a service is not functional,
and may react by creating new instances or issue notices to an alerting system.

### Configuration Handling ###

For any component in a service requiring configuration, it can simply get a `ServiceProperties`
object injected into it, and request the configuration properties from it. Properties are
located from three locations: command-line, environment variables, and configuration plugin.
They are applied in that order. The same property with a different value from a later source
overwrites the earlier source. The configuration plugin provides the ability to do long-polling so 
that each service can get real-time updates to changes of the configuration from arbitrary sources. 
There are hooks for service components to get real-time notifications of configuration changes.

### Logging ###

There is standardized logging in place. The log records are json objects, and `Marker`s can
be used to dynamically create properties on those objects. The log format can be completely customized.

### Metrics ###

There is standardized metric handling in place. We use Micrometer as a metric library,
which has plugins for dozens of back-end systems to integrate with.

## Event-driven Architecture ###

There are factories/builders in place to easily create publishers and subscribers that 
integrate with Kafka or AMQP (ActiveMQ, RabbitMQ, etc).

### Output Artifacts ###

One can build a shadow jar (fat jar - all dependencies included), or an OCI image. One might
use the shadow jar for developer testing or publishing as an artifact.

### Database Migrations ###

There is currently support for Flyway database migrations, which supports many different
SQL databases. This support extends into carefully controlling the service lifecycle
and readiness probes.

### Dependency Injection ###

Dependency injection is heavily used in Ja-micro. It is strictly supporting Guice.

### Service Testing ###

The ability to test your microservice code from several different aspects and to achieve
various goals is driven by the testing pyramid approach.  At the base of the pyramid is where
the majority of your tests should be, which are unit tests.  They are at the base because 
this is usually the first opportunity to catch bugs and run the fastest.  Up on level from
here is what we call Integration Tests.  Here, you are testing specific integration code,
such as a PostgresRepository.  In our recipes, we show an example of how to test database
migration code and repository code against a real instance of Postgres running in a container.
Going up another level, we have Service Integration Tests, which boots the compiled
service as a container along with other required infrastructure components in docker-compose.
To eliminate the problem that would arise of starting containers for every other service
dependency (and their dependencies, etc.), there exists a class called `ServiceImpersonator`
that can serve as a complete service mock, serving real rpc requests. However, the developer 
maps the requests and responses instead of a real instance serving those requests.
