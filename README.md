**Note**: Original repository here [https://github.com/deviantony/docker-elk](https://github.com/deviantony/docker-elk) 

# ElasticSearch Cluster and Kibana on Docker

Run the latest version of the Elastic stack with Docker and Docker Compose.

It will give you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch and the visualization power of Kibana.

Based on the official Docker images from Elastic:

* [elasticsearch](https://github.com/elastic/elasticsearch-docker)
* [kibana](https://github.com/elastic/kibana-docker)

## Contents

1. [Requirements](#requirements)
   * [Host setup](#host-setup)
2. [Usage](#usage)
   * [Bringing up the stack](#bringing-up)
3. [Configuration](#configuration)
   * [How can I tune the Kibana configuration?](#how-can-i-tune-the-kibana-configuration)
   * [How can I tune the Elasticsearch configuration?](#how-can-i-tune-the-elasticsearch-configuration)
4. [Storage](#storage)
5. [How can I add plugins](#extensibility)
6. [JVM tuning](#jvm-tuning)
7. [Going further](#going-further)
   * [Using a newer stack version](#using-a-newer-stack-version)

## Requirements
### Host Setup
1. Install [Docker](https://www.docker.com/community-edition#/download)
2. Install [Docker Compose](https://docs.docker.com/compose/install/)
3. Clone this repository


#### [Virtual memory](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html#vm-max-map-count)
Elasticsearch uses a mmapfs directory by default to store its indices. The default operating system limits on mmap counts is likely to be too low, which may result in out of memory exceptions.
On Linux, you can increase the limits by running the following command as root:

```sudo sysctl -w vm.max_map_count=262144```

To set this value permanently, update the vm.max_map_count setting in /etc/sysctl.conf. To verify after rebooting, run sysctl vm.max_map_count.
The RPM and Debian packages will configure this setting automatically. No further configuration is required.

#### [Disable all swap files](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html#disable-swap-files)
Usually Elasticsearch is the only service running on a box, and its memory usage is controlled by the JVM options. There should be no need to have swap enabled.
On Linux systems, you can disable swap temporarily by running:

 ```sudo swapoff -a```

To disable it permanently, you will need to edit the /etc/fstab file and comment out any lines that contain the word swap.

####[Configure swappiness](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html#swappiness)
Another option available on Linux systems is to ensure that the sysctl value vm.swappiness is set to 1. 


```sudo sysctl -w vm.swappiness=1```

This reduces the kernel’s tendency to swap and should not lead to swapping under normal circumstances, while still allowing the whole system to swap in emergency conditions.

## Configuration

**NOTE**: Configuration is not dynamically reloaded, you will need to restart docker-compose after any change in the
configuration of a component.

### How can I tune the Kibana configuration?

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

### How can I tune the Elasticsearch configuration?

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

## Usage

### Bringing up

Start using `docker-compose`:

```console
$ docker-compose up
```

You can also run all services in the background (detached mode) by adding the `-d` flag to the above command.

Give Kibana a few seconds to initialize, then access the Kibana web UI by hitting
[http://localhost:5601](http://localhost:5601) with a web browser.

By default, the stack exposes the following ports:
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

## Storage

The data stored in Elasticsearch will be persisted after container reboot but not after container removal.

## Extensibility

To add plugins to Elasticsearch or Kibana component you have to:

1. Add a `RUN` statement to the corresponding `Dockerfile` (eg. `RUN kibana-plugin install x-pack`)
2. Add the associated plugin code configuration to the service configuration
3. Rebuild the images using the `docker-compose build` command

## JVM tuning

### How can I specify the amount of memory used by a service?

By default, both Elasticsearch start with [1/4 of the total host
memory](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size) allocated to
the JVM Heap Size.

The startup scripts for Elasticsearch can append extra JVM options from the value of an environment
variable, allowing the user to adjust the amount of memory that can be used by each component:

| Service       | Environment variable |
|---------------|----------------------|
| Elasticsearch | ES_JAVA_OPTS         |

To accomodate environments where memory is scarce (Docker for Mac has only 2 GB available by default), the Heap Size
allocation is capped by default to 512MB per service in the `docker-compose.yml` file. If you want to override the
default JVM configuration, edit the matching environment variable(s) in the `docker-compose.yml` file.

For example, to increase the maximum JVM Heap Size for Elasticsearch:

```yml
 elasticsearch:

  environment:
    ES_JAVA_OPTS: "-Xmx2g -Xms2g"
```

## Going further

### Using a newer stack version

To use a different Elastic Stack version than the one currently available in the repository, simply change the version
number inside the `.env` file, and rebuild the stack with:

```console
$ docker-compose build
$ docker-compose up
```

**NOTE**: Always pay attention to the [upgrade instructions](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html)
for each individual component before performing a stack upgrade.
