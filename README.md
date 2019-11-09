# Ja-micro [![License](https://img.shields.io/:license-apache-blue.svg)](https://opensource.org/licenses/Apache-2.0) 

Ja-micro is a lightweight Java framework for building microservices.

## Introduction ##

Ja-micro is a framework that allows developers to easily develop microservices in 
Java. It was developed beginning in 2016, during a push to create a new platform 
based on Kubernetes. With version 4, the focus is on easy integration with OpenShift, 
and using standards such as OpenApi, gRPC and AMQP.  It also supports database 
migrations, Infinispan distributed memory featues, and a rich Kafa integraions.  It 
includes a Gradle wrapper for elimination of boilerplate and simultaneously rich 
customization of build pipelines. 

The framework takes care of many of the concerns needed in a modern containerized 
application so that developers can simply focus on the business functionality 
of their services rather than tedius boostrapping.  Cloud native principles 
are used wherever possible, and example code explores other best practices.

## Features ##

* Simply build service an OCI container or fat jar.
* Configuration from environment, command-line and external configuration services.
* Standardized json logging.
* Standardized metrics reporting
* Simple interface for calling endpoints on other services and handling errors from them.
* Simple interface for a service to support readiness and health checks.
* Database migrations built-in.
* Simplified event-handling using Kafka or AMQP.
* Guice dependency injection for ease of implementation and testing scenarios.
* Components to create service integration (contract) testing scenarios.

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

The `RpcCallException` class defines all of
our exception categories, their default retriable setting, and what the resulting HTTP status code is (when using
an HTTP transport). When one service calls another (client calling server), if a server throws an exception
during processing the response, the exception is transparently transported back to the client and can be
rethrown on the client-side (ignoring the retry aspect here).

Default RpcClient retry policy (1) can be overridden with setting 'rpcClientRetries'.
Default RpcClient timeout policy (1000ms) can be overridden with setting 'rpcClientTimeout'.

### Health Checks ###

Every few seconds, the health state for a service instance is reported to the service
registry plugin. Using annotations, there is support for a service developer to easily
hook into health checking to mark a service instance as unhealhty. There is also an
interface to immediately change the health state of an instance.

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

There is standardized metric handling in place. This is heavily opinionated to Sixt's
infrastructure, and uses metrics formatted in a specific format and sent to an influx agent
to be reported back to a central influxdb cluster.  The metrics reporting is pluggable to 
support reporting metrics in any desired format to any desired destination.

### Kafka Events ###

There are factories/builders in place to easily create publishers and subscribers for
topics in Kafka.

### Output Artifacts ###

One can build a shadow jar (fat jar - all dependencies included), or a docker image. One might
use the shadow jar for developer testing. The shadow jar is a dependency of the docker image
tasks as well. To start a service in a debugger, use the `JettyServiceBase` as a main class.

### Database Migrations ###

There is currently support for Flyway database migrations, which supports many different
SQL databases. This support extends into carefully controlling the service lifecycle
and health checks. A future version should support basic migration support for DynamoDB
instances.

### Dependency Injection ###

Dependency injection is heavily used in Ja-micro. It is strictly supporting Guice.

### Service Integration Testing ###

We heavily use automation at Sixt in our microservice projects. To support this, the framework 
gives the ability to developers to automate service integration tests. What this means is 
that core infrastructure dependencies (for example, on one service, this is consul, 
postgres, zookeeper and kafka) and the
service itself are started as containers under docker-compose. Additionally, to
eliminate the problem that would arise of starting containers for every other service
dependency (and their dependencies, etc.), there exists a class called `ServiceImpersonator`
that can serve as a complete service mock, available in service registry and serving
real rpc requests. However, the developer maps the requests and responses instead
of a real instance serving those requests.

