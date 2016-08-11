# Amalgam8 Registry

[![GoReportCard Widget]][GoReportCard] [![Travis Widget]][Travis]

[GoReportCard]: https://goreportcard.com/report/github.com/amalgam8/registry
[GoReportCard Widget]: https://goreportcard.com/badge/github.com/amalgam8/registry
[Travis]: https://travis-ci.org/amalgam8/registry
[Travis Widget]: https://travis-ci.org/amalgam8/registry.svg?branch=master

The Amalgam8 Registry is a multi-tenant, highly-available service for service
registration and service discovery in microservice applications.  For high
availability, run the Registry in clustered mode. In clustered mode, the
Registry provides eventual consistency for data synchronization across
Registry instances.

The Registry is built using an extensible catalog model that allows the user
to update the Amalgam8 Registry with information stored in other service
registries such as Kubernetes, Consul, etc. In addition, the Registry can
be used as a drop-in replacement for
[Netflix Eureka](https://github.com/Netflix/eureka) with full API
compatibility. Eureka compatibility enables the Registry to be used with
Java clients using [Netflix Ribbon](https://github.com/Netflix/ribbon).

By default, the Registry operates without any authentication. It also
supports two authentication mechanisms: a trusted auth mode for local
testing and development, and a JWT auth mode for production deployments.

See https://www.amalgam8.io for an overview of the Amalgam8 project
and https://www.amalgam8.io/docs for detailed documentation.


## Usage

To get started, use the current stable version of the Registry from Docker
Hub.

```bash
docker run amalgam8/a8-registry:latest -auth_mode=trusted
```

### Command Line Flags and Environment Variables

The Amalgam8 Registry supports a number of configuration options,
most of which are set through environment variables. The environment
variables can be set via command line flags as well.

The following environment variables are available. All of them are optional.

#### Registry Configuration

| Environment Key | Flag Name                   | Description | Default Value |
|:----------------|:----------------------------|:------------|:--------------|
| `API_PORT` | `--api_port` | API port number | 8080 |
| `LOG_LEVEL` | `--log_level` | Logging level. Supported values are: `debug`, `info`, `warn`, `error`, `fatal`, `panic` | `debug` |
| `LOG_FORMAT` | `--log_format` | Logging format. Supported values are: `text`, `json`, `logstash` | `text` |
| `NAMESPACE_CAPACITY` | `--namespace_capacity` | maximum number of instances that may be registered in a namespace | -1 (no capacity limit) |  
| `DEFAULT_TTL` | `--default_ttl` | Registry default instance time-to-live (TTL) | 30s |
| `MIN_TTL` | `--min_ttl` | Minimum TTL that may be specified during registration | 10s | 
| `MAX_TTL` | `--max_ttl` | Maximum TTL that may be specified during registration | 10m |


#### Authentication and Authorization

The Amalgam8 Registry supports multi-tenancy by isolating each tenant into a separate namespace.
A namespace is defined by an opaque string carried in the HTTP `Authorization` header of API requests. The following
namespace authorization methods are supported and controlled via the `AUTH_MODE` environment variable (or `--auth_mode`
flag):
* None: if no authorization mode is defined, all instances are registered into a default shared namespace. 
* Trusted: namespace is retrieved directly from the Authorization header. This provides namespace separation in a trusted
environment (e.g., single tenant with multiple applications or environments).
* JWT: encodes the namespace value in a signed JWT token claim. 

| Environment Key | Flag Name                   | Description | Default Value |
|:----------------|:----------------------------|:------------|:--------------|
| `AUTH_MODE` | `--auth_mode` | Authentication modes. Supported values are: `trusted`, `jwt` | none (no isolation) |
| `JWT_SECRET` | `--jwt_secret` | Secret key for JWT authentication | none (must be set if `AUTH_MODE` is `jwt`) |
| `REQUIRE_HTTPS` | `--require_https` | Require clients to use HTTPS for API calls | `false` |

If `jwt` is specified, `JWT_SECRET` (or `--jwt_secret`) must be set as well to allow encryption and decryption.
Namespace value encoding must be present in every API call using HTTP Bearer Authorization:

```bash
Authorization: Bearer jwt.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ...wifQ.Gbz4G_O...NqdY`
```

#### Clustering

Amalgam8 Registry uses a memory only storage solution, without persistency (although different storage 
backends can be implemented). To provide HA and scale, the Registry can be run in a cluster and supports replication
between cluster members.

Peer discovery currently uses a shared volume between all members. The
volume must be mounted RW into each container.  We are exploring
alternative discovery mechanisms.


| Environment Key | Flag Name                   | Description | Default Value |
|:----------------|:----------------------------|:------------|:--------------|
| `CLUSTER_SIZE` | `--cluster_size` | Cluster minimal healthy size, peers detecting a lower value will log errors | 1 (standalone) |
| `CLUSTER_DIR` | `--cluster_dir` | Filesystem directory for cluster membership | none, must be specified for clustering to work |
| `REPLICATION` | `--replication` | Enable replication between cluster members | `false` |
| `REPLICATION_PORT` | `--replication_port` | Replication port number | 6100 |
| `SYNC_TIMEOUT` | `--sync_timeout` | Timeout for establishing connections to peers for replication | 30s |

#### Catalog Extensions

The Amalgam8 Registry supports read-only catalogs extensions. 
The content of each catalog extension (e.g., Kubernetes, Docker-Swarm, FileSystem, etc) is read by the Registry and
returned to the user along with the content of the Registry itself.

| Environment Key | Flag Name                   | Description | Default Value |
|:----------------|:----------------------------|:------------|:--------------|
| `K8S_URL` | `--k8s_url` | Enable kubernetes catalog and specify the API server | (none) |
| `K8S_TOKEN` | `--k8s_token` | Kubernetes API token | (none) |
| `FS_CATALOG` | `--fs_catalog` | Enable FileSystem catalog and specify the directory of the config files. The format of the file names in the directory should be `<namespace>.conf`. See [FileSystem catalog documentation](doc/filesystem_catalog.md) for more information | (none) |


## REST API

Amalgam8 Registry [API documentation](https://amalgam8.io/registry) is
available in Swagger format.

## Building from source

To build from source, clone this repository, and follow the instructions below.

### Pre-requisites

* Docker engine >=1.10
* Go toolchain (tested with 1.6.x). See [Go downloads](https://golang.org/dl/) and [installation instructions](https://golang.org/doc/install).


### Building a Docker Image

To build the docker image for the Amalgam8 Registry service, run the
following commands:

```bash
cd $GOPATH/src/github.com/amalgam8/registry
make build docker
```

You should now have a docker image tagged `a8-registry:latest`.

### Building an Executable

The Amalgam8 Registry can also be run outside of a docker container as a Go
binary.  This is not recommended for production, but it can be useful for
development or easier integration with your local Go tools.

The following commands will build and run it as a Go binary:

```
cd $GOPATH/src/github.com/amalgam8/registry
make build
./bin/registry
```

### Makefile Targets

The following Makefile targets are available.

| Make Target      | Description |
|:-----------------|:------------|
| `build`          | *(Default)* `build` builds the Registry binary in the ./bin directory |
| `precommit`      | `precommit` should be run by developers before committing code changes. It runs code formatting and checks. |
| `test`           | `test` runs (short duration) tests using `go test`. You may also `make test.all` to include long running tests. |
| `docker`         | `docker` packages the binary in a docker container |
| `release`        | `release` builds a tarball with the Registry binary |
| `clean`          | `clean` removes build artifacts. *Note: this does not remove docker images* |

### Continuous Integration with Travis CI

Continuous builds are run on Travis CI. These builds use the `.travis.yml` configuration.

## Release Workflow

This section includes instructions for working with releases, and is intended for the project's maintainers (requires write permissions)

### Creating a release

1.  Set a version for the release, by incrementing the current version
    according to the [semantic versioning](https://semver.org/)
    guidelines. For example,

    ```bash
    export VERSION=v0.1.0
    ```

1.  Update the APP_VER variable in the Makefile such that it matches with
    the VERSION variable above.

1.  Create an [annotated tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging#Annotated-Tags) in your local copy of the repository:
   
    ```bash
    git tag -a -m "Release $VERSION" $VERSION [commit id]
    ```

    The `[commit id]` argument is optional. If not specified, HEAD is used.
   
1.  Push the tag back to the Amalgam8 upstream repository on GitHub:

    ```bash
    git push upstream $VERSION
    ```
   This command automatically creates a release object on GitHub, corresponding to the pushed tag.
   The release contains downloadable packages of the source code (both as `.zip` and `.tag.gz` archives).

1.  Edit the `CHANGELOG.md` file, describing the changes included in this release.

1.  Edit the [GitHub release object](https://github.com/amalgam8/registry/releases), and add a title and description (according to `CHANGELOG.md`).

## License
Copyright 2016 IBM Corporation

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## Contributing

Contributions and feedback are welcome! 
Proposals and pull requests will be considered. 
Please see the [CONTRIBUTING.md](https://github.com/amalgam8/amalgam8.github.io/blob/master/CONTRIBUTING.md) file for more information.
