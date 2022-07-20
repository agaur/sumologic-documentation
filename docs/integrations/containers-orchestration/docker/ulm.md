---
id: ulm
title: Docker ULM
sidebar_label: Docker ULM
---


The Sumo Logic App for Docker ULM is a unified logs and metrics app that allows you to monitor your Docker deployment. The preconfigured dashboards present information about container state and resource usage, including CPU, memory, block I/O, and network.


This app has been tested with Docker v8.03.1-ce and Docker API v1.37.

Log and metric types

Sumo’s Docker Logs source and Docker Stats source use the Docker Engine API to gather the following data from Docker:

* Docker container logs. Sumo’s Docker Logs source collects container logs. For information about the API Sumo uses to collect logs, see the [https://docs.docker.com/engine/api/v1.37/#operation/ContainerLogs](https://docs.docker.com/engine/api/v1.37/#operation/ContainerLogs) in Docker help.
* Docker engine events. Sumo’s Docker Log source collect Docker events. For information about Docker events, see [https://docs.docker.com/engine/api/v1.37/#operation/SystemPing](https://docs.docker.com/engine/api/v1.37/#operation/SystemPing) in Docker help.
* Docker container stats. Sumo’s Docker Stats source collects stats. For information about Docker stats, see [https://docs.docker.com/engine/api/v1.37/#operation/ContainerStats](https://docs.docker.com/engine/api/v1.37/#operation/ContainerStats) in Docker help.

By default, you can monitor up to 40 Docker containers on a Docker host. If you want to monitor more than 40 containers on a given host you can configure a larger number in `collector.properties`, as described in [Collect Logs and Metrics for Docker ULM](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM). We don’t support monitoring more than 100 containers on a Docker host.


## Collect Logs and Metrics for Docker ULM

Docker is a lightweight open platform that provides a way to package applications in containers for a software development environment.


Windows operating systems are not supported.

You can add the following types of Docker Sources to an Installed Collector on Linux:

* **Docker Logs. **Collects stdout/stderr logs from processes that are running within Docker containers.
* **Docker Stats. **Collects metrics about Docker containers.


There are alternative methods for collecting Docker logs and metrics. See [Docker Collection Methods](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Docker_Collection_Methods) for more information.


### Notes regarding Docker Sources



* We recommend running the Sumo collector on the same host where you run Docker. You can deploy the collector as a Docker container. For more information, see [https://hub.docker.com/r/sumologic/collector/](https://hub.docker.com/r/sumologic/collector/).
* The Sumo Logic Collector uses the Docker Remote API to collect Docker logs. This requires that the log driver configured on the container uses either the `json-file` or `journald` option, as described in [https://docs.docker.com/engine/admin/logging/overview/](https://docs.docker.com/engine/admin/logging/overview/).
* If you are using Kubernetes we recommend using an open source [Kubernetes Fluentd plugin](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Kubernetes_Fluentd_plugin).
* The Docker Log Source uses timestamps from logs to track collection. You need to ensure your log format has a [well-defined timestamp](https://help.sumologic.com/03Send-Data/Sources/04Reference-Information-for-Sources/Timestamps%2C-Time-Zones%2C-Time-Ranges%2C-and-Date-Formats) and the Source is configured to detect it properly. If there are issues with timestamp detection and the Docker container is restarted the Source will reingest all log data since there are no timestamps to track.
* Docker truncates logs at 16kb and Sumo Logic does not stitch them back together.


### Add a Docker Logs Source

1. In the Sumo web app, select **Manage Data > Collection > Collection**.
2. Navigate to the collector you installed on the Docker host, and select **Add > Add Source**.
3. Select **Docker Logs**. The Docker Logs page appears.

4. Configure the source fields:
    1. **Name**. (Required).
    2. **Description**. (Optional).
    3. **URI**. Enter the URI of the Docker daemon.
        * If your collector runs on the same host as the Docker containers it will monitor, enter the non-networked Unix socket:     `unix:///var/run/docker.sock`
        * If your collector runs on a different machine than the Docker host, you can determine its URI from a Docker environment variable. Run the `docker-machine` command to find the Docker environment variables. The command's syntax is `$ docker-machine env machine-name`. \
For example,

```
$ docker-machine env default \
export DOCKER_TLS_VERIFY="1" \
export DOCKER_HOST="tcp://192.168.99.100:2376" \
export DOCKER_CERT_PATH="/Users/sumo/.docker/machine/machines/default" \
export DOCKER_MACHINE_NAME="default" \
# Run this command to configure your shell:  \
# eval "$(docker-machine env default)" \
```

Take the value of the `DOCKER_HOST` variable, change "tcp" to "https", and enter that value as the URI. For example, `https://192.168.99.100:2376`.
    4. **Cert Path**. (Required for remote access only) Enter the path to the certificate files on the local machine where the collector runs. In the example above, the cert path is: `/Users/sumo/.docker/machine/machines/default`.
    5. **Event Logs**. Check this box to collect Docker events as well as standard Docker logs.
    6. **Collect From** and **Container Filters**. If you want to collect from all containers, click the **All Containers** radio button. If you want to collect from selected containers, click the **Specified Container Filters** radio button, and specify filter expressions in the **Container Filters** field. For information about how to define container filters, see [more about defining container filters](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM#More_about_defining_container_filters-851) below.
        * By default, you can collect from up to 40 containers. To increase the limit, stop the Collector service, edit the `collector.properties` file (in the `config` subdirectory of the collector installation directory), and add the `docker.maxPerContainerConnections` property. The maximum supported value is 100. Then start the Collector service. See [collector.properties](https://help.sumologic.com/03Send-Data/Installed-Collectors/05Reference-Information-for-Collector-Installation/collector.properties) for details on modifying this configuration file.
    7. **Source Host**. Enter the hostname or IP address of the source host. If not specified, it’s assumed that the host is the machine where Docker is running. The hostname can be a maximum of 128 characters.  \
If desired, you can use Docker variables to construct the Source Host value. For more information, see [Configure sourceCategory and sourceHost using variables.](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM#Configure_sourceCategory_and_sourceHost_using_variables-851)
    8. **Source Category**. Enter a string used to tag the output collected from this Source with searchable metadata. For example, typing **web_apps** tags all the logs from this Source in the sourceCategory field, so running a search on **_sourceCategory=web_apps** would return logs from this Source. For more information, see [Metadata Naming Conventions](https://help.sumologic.com/03Send-Data/Sources/04Reference-Information-for-Sources/Metadata-Naming-Conventions) and our [Best Practices: Good Source Category, Bad Source Category](https://help.sumologic.com/03Send-Data/01-Design-Your-Deployment/Best-Practices%3A-Good-Source-Category%2C-Bad-Source-Category). \
If desired, you can use Docker variables to construct the Source Category value. For more information, see [Configure sourceCategory and sourceHost using variables.](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM#Configure_sourceCategory_and_sourceHost_using_variables-851)
    9. **Fields**. Click the **+Add Field** link to add custom log metadata [Fields](https://help.sumologic.com/Manage/Fields).
        * Define the fields you want to associate, each field needs a name (key) and value.

 A green circle with a check mark is shown when the field exists and is enabled in the Fields table schema.

 An orange triangle with an exclamation point is shown when the field doesn't exist, or is disabled, in the Fields table schema. In this case, an option to automatically add or enable the nonexistent fields to the Fields table schema is provided. If a field is sent to Sumo that does not exist in the Fields schema or is disabled it is ignored, known as dropped.
5. Configure the Advanced options.
    10. **Enable Timestamp Parsing**. This option is checked by default and **required**. See the [Notes section](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM#Notes_regarding_Docker_Sources-851) above for details.
    11. **Time Zone**. Default is “Use time zone from log file”.
    12. **Timestamp Format**. Default is “Automatically detect the format”.
    13. **Encoding**. Default is “UTF-8”.
    14. **Enable Multiline Processing. **The** _Detect messages spanning multiple lines_** option is checked by default. See [Collecting Multiline Logs](https://help.sumologic.com/03Send-Data/Sources/04Reference-Information-for-Sources/Collecting-Multiline-Logs) for details on multiline processing and its options.
        * **Infer Boundaries**. This option is checked by default.
        * **Boundary Regex**. If multiple processing is enabled, and **Infer Boundaries** is disabled, enter a regular expression for message boundaries.
6. Configure processing rules. For more information, see** **[Processing Rules](https://help.sumologic.com/Manage/Collection/Processing-Rules).


### Add a Docker Stats Source

1. In Sumo select **Manage Data > Collection > Collection**.
2. Navigate to the collector you installed on the Docker host, and select **Add > Add Source**.
3. Select **Docker Stats.** The following Docker Stats page appears. There are two possible content types available, select **Metrics** to collect data as metrics, or select **Logs** (JSON) to collect data as JSON logs.


8
To collect metrics for the Docker ULM App, select **Metrics** as the Content Type.


9




1. Configure the source fields:
    1. **Name.** (Required)
    2. **Description. **(Optional)
    3. **URI**. Enter the URI of the Docker daemon.
        * If your collector runs on the same host as the Docker containers it will monitor, enter the non-networked Unix socket: \
`unix:///var/run/docker.sock`
        * If your collector runs on a different machine than the Docker host, you can determine its URI from a Docker environment variable. Run the `docker-machine` command to find the Docker environment variables. The command's syntax is `$ docker-machine env machine-name`. \
For example, \
 \
`$ docker-machine env default \
export DOCKER_TLS_VERIFY="1" \
export DOCKER_HOST="tcp://192.168.99.100:2376" \
export DOCKER_CERT_PATH="/Users/sumo/.docker/machine/machines/default" \
export DOCKER_MACHINE_NAME="default" \
# Run this command to configure your shell:  \
# eval "$(docker-machine env default)" \
 \
`Take the value of the `DOCKER_HOST` variable, change "tcp" to "https", and enter that value as the URI. For example, `https://192.168.99.100:2376`.
    4. **Cert Path**. (Required for remote access only) Enter the path to the certificate files on the local machine where the collector runs. In the example above, the cert path is: /Users/sumo/.docker/machine/machines/default
    5. **Collect From** and **Container Filters**. If you want to collect from all containers, click the **All Containers** radio button. If you want to collect from selected containers, click the **Specified Container Filters** radio button, and specify filter expressions in the **Container Filters** field. For information about how to define container filters, see [more about defining container filters](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM#More_about_defining_container_filters-851) below.
        * By default, you can collect from up to 40 containers. To increase the limit, stop the Collector service, edit the `collector.properties` file (in the `config` subdirectory of the collector installation directory), and add the `docker.maxPerContainerConnections` property. The maximum supported value is 100. Then start the Collector service. See [collector.properties](https://help.sumologic.com/03Send-Data/Installed-Collectors/05Reference-Information-for-Collector-Installation/collector.properties) for details on modifying this configuration file.
    6. **Source Host**. Enter the hostname or IP address of the source host. If not specified, it’s assumed that the host is the machine where Docker is running. The hostname can be a maximum of 128 characters. If desired, you can use Docker variables to construct the Source Host value. For more information, see [Configure sourceCategory and sourceHost using variables.](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM#Configure_sourceCategory_and_sourceHost_using_variables-851)
    7. **Source Category**. Enter a string used to tag the output collected from this Source with searchable metadata. For example, typing **web_apps** tags all the logs from this Source in the sourceCategory field, so running a search on **_sourceCategory=web_apps** would return logs from this Source. For more information, see [Metadata Naming Conventions](https://help.sumologic.com/03Send-Data/Sources/04Reference-Information-for-Sources/Metadata-Naming-Conventions) and our [Best Practices: Good Source Category, Bad Source Category](https://help.sumologic.com/03Send-Data/01-Design-Your-Deployment/Best-Practices%3A-Good-Source-Category%2C-Bad-Source-Category). \
If desired, you can use Docker variables to construct the Source Host value. For more information, see [Configure sourceCategory and sourceHost using variables.](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Docker_Apps/Docker_ULM/Collect_Logs_and_Metrics_for_Docker_ULM#Configure_sourceCategory_and_sourceHost_using_variables-851)
    8. **Fields**. Click the **+Add Field** link to add custom metric metadata. Define the fields you want to associate, providing a name (key) and value for each.
    9. **Scan Interval**. This option sets how often the source is scanned. Setting a shorter frequency increases message volume, and can cause your deployment to incur additional charges. The minimum acceptable scan interval is 1 second.
    10. **Metrics** (Available if content type selected is Metrics). Select the Docker metrics you want to be ingested, see Docker metrics definitions for details.


### Docker metrics definitions


<table>
  <tr>
   <td><strong>Metrics Name</strong>
   </td>
   <td><strong>Unit</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong>CPU</strong>
   </td>
  </tr>
  <tr>
   <td>cpu_percentage
   </td>
   <td>Percent
   </td>
   <td>Percentage of CPU used
   </td>
  </tr>
  <tr>
   <td>online_cpus
   </td>
   <td>Count
   </td>
   <td>Number of CPUs online (only available on API v1.27 or higher)
   </td>
  </tr>
  <tr>
   <td>system_cpu_usage
   </td>
   <td>Nanoseconds
   </td>
   <td>Host’s cumulative CPU usage
   </td>
  </tr>
  <tr>
   <td>cpu_usage.percpu_usage
   </td>
   <td>Nanoseconds
   </td>
   <td>Respective usage for every CPU (scalable with cpu=[0..x])
   </td>
  </tr>
  <tr>
   <td>cpu_usage.total_usage
   </td>
   <td>Nanoseconds on Linux 100's of nanoseconds on Windows
   </td>
   <td>Total CPU time consumed
   </td>
  </tr>
  <tr>
   <td>cpu_usage.usage_in_kernelmode
   </td>
   <td>Nanoseconds
   </td>
   <td>CPU usage in the kernel mode
   </td>
  </tr>
  <tr>
   <td>cpu_usage.usage_in_usermode
   </td>
   <td>Nanoseconds
   </td>
   <td>CPU usage in the user mode
   </td>
  </tr>
  <tr>
   <td>throttling_data.periods
   </td>
   <td>Count
   </td>
   <td>Number of enforcement intervals that have elapsed
   </td>
  </tr>
  <tr>
   <td>throttling_data.throttled_periods
   </td>
   <td>Count
   </td>
   <td>Number of enforcement intervals that have been throttled
   </td>
  </tr>
  <tr>
   <td>throttling_data.throttled_time
   </td>
   <td>Nanoseconds
   </td>
   <td>Total CPU time has been throttled
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong>Memory</strong>
   </td>
  </tr>
  <tr>
   <td>failcnt
   </td>
   <td>Count
   </td>
   <td>Times of memory usage hit the limits
   </td>
  </tr>
  <tr>
   <td>limit
   </td>
   <td>Bytes
   </td>
   <td>Memory limit set on the container (or on the host if it's not set on container)
   </td>
  </tr>
  <tr>
   <td>max_usage
   </td>
   <td>Bytes
   </td>
   <td>Maximum of memory usage
   </td>
  </tr>
  <tr>
   <td>memory_percentage
   </td>
   <td>Percent
   </td>
   <td>Percentage of memory used
   </td>
  </tr>
  <tr>
   <td>usage
   </td>
   <td>Bytes
   </td>
   <td>Current memory usage
   </td>
  </tr>
  <tr>
   <td>stats.active_anon
   </td>
   <td>Bytes
   </td>
   <td>The anonymous memory that has been identified as active by the kernel
   </td>
  </tr>
  <tr>
   <td>stats.active_file
   </td>
   <td>Bytes
   </td>
   <td>The file cache memory that has been identified as active by the kernel
   </td>
  </tr>
  <tr>
   <td>stats.cache
   </td>
   <td>Bytes
   </td>
   <td>The memory used by the processes of this cgroup that can be associated precisely with a block on a block device
   </td>
  </tr>
  <tr>
   <td>stats.hierarchical_memory_limit
   </td>
   <td>Bytes
   </td>
   <td>The hierarchical memory limit
   </td>
  </tr>
  <tr>
   <td>stats.inactive_anon
   </td>
   <td>Bytes
   </td>
   <td>he anonymous memory that has been identified as inactive by the kernel
   </td>
  </tr>
  <tr>
   <td>stats.inactive_file
   </td>
   <td>Bytes
   </td>
   <td>The file cache memory that has been identified as inactive by the kernel
   </td>
  </tr>
  <tr>
   <td>stats.mapped_file
   </td>
   <td>Bytes
   </td>
   <td>The cache memory mapped by the processes in the cgroup
   </td>
  </tr>
  <tr>
   <td>stats.pgfault
   </td>
   <td>Count
   </td>
   <td>The number of times that a process of the cgroup triggered a 'page fault'
   </td>
  </tr>
  <tr>
   <td>stats.pgmajfault
   </td>
   <td>Count
   </td>
   <td>The number of times that a process of the cgroup triggered a “major fault”
   </td>
  </tr>
  <tr>
   <td>stats.pgpgin
   </td>
   <td>Count
   </td>
   <td>The number of times that the system has paged in from disk
   </td>
  </tr>
  <tr>
   <td>stats.pgpgout
   </td>
   <td>Count
   </td>
   <td>The number of times that the system has paged out to disk
   </td>
  </tr>
  <tr>
   <td>stats.rss
   </td>
   <td>Bytes
   </td>
   <td>The memory that does not correspond to anything on disk: stacks, heaps, or anonymous memory maps
   </td>
  </tr>
  <tr>
   <td>stats.rss_huge
   </td>
   <td>Bytes
   </td>
   <td>The anonymous memory transparent hugepages
   </td>
  </tr>
  <tr>
   <td>stats.unevictable
   </td>
   <td>Bytes
   </td>
   <td>The memory that cannot be reclaimed
   </td>
  </tr>
  <tr>
   <td>stats.writeback
   </td>
   <td>Bytes
   </td>
   <td>File anonymous cache that are queued for syncing to disk
   </td>
  </tr>
  <tr>
   <td>stats.total_active_anon
   </td>
   <td>Bytes
   </td>
   <td>The anonymous memory that has been identified has respectively active by the kernel (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_active_file
   </td>
   <td>Bytes
   </td>
   <td>The file cache memory that has been identified has respectively active by the kernel (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_cache
   </td>
   <td>Bytes
   </td>
   <td>The memory used by the processes of this cgroup that can be associated precisely with a block on a block device (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_inactive_anon
   </td>
   <td>Bytes
   </td>
   <td>The anonymous memory that has been identified has respectively inactive by the kernel (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_inactive_file
   </td>
   <td>Bytes
   </td>
   <td>The file cache memory that has been identified has respectively inactive by the kernel (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_mapped_file
   </td>
   <td>Bytes
   </td>
   <td>The cache memory mapped by the processes in the control group (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_pgfault
   </td>
   <td>Count
   </td>
   <td>The number of times that a process of the cgroup triggered a 'page fault' (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_pgmajfault
   </td>
   <td>Count
   </td>
   <td>The number of times that a process of the cgroup triggered a “major fault” (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_pgpgin
   </td>
   <td>Count
   </td>
   <td>The number of times that the system has paged in from disk (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_pgpgout
   </td>
   <td>Count
   </td>
   <td>The number of times that the system has paged out to disk (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_rss
   </td>
   <td>Bytes
   </td>
   <td>The memory that doesn’t correspond to anything on disk: stacks, heaps, and anonymous memory maps (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_rss_huge
   </td>
   <td>Bytes
   </td>
   <td>The anonymous memory transparent hugepages (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_unevictable
   </td>
   <td>Bytes
   </td>
   <td>The memory that cannot be reclaimed (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td>stats.total_writeback
   </td>
   <td>Bytes
   </td>
   <td>File anonymous cache that are queued for syncing to disk (includes sub-cgroups)
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong>Network - Scalable on API v1.21 or higher, with dimension "interface"</strong>
   </td>
  </tr>
  <tr>
   <td>rx_bytes
   </td>
   <td>Bytes
   </td>
   <td>Data volume (received)
   </td>
  </tr>
  <tr>
   <td>rx_dropped
   </td>
   <td>Count
   </td>
   <td>Number of dropped packets (received)
   </td>
  </tr>
  <tr>
   <td>rx_errors
   </td>
   <td>Count
   </td>
   <td>Number of error packets (received)
   </td>
  </tr>
  <tr>
   <td>rx_packets
   </td>
   <td>Count
   </td>
   <td>Number of total packets (received)
   </td>
  </tr>
  <tr>
   <td>tx_bytes
   </td>
   <td>Bytes
   </td>
   <td>Data volume (transmitted)
   </td>
  </tr>
  <tr>
   <td>tx_dropped
   </td>
   <td>Count
   </td>
   <td>Number of dropped packets (transmitted)
   </td>
  </tr>
  <tr>
   <td>tx_errors
   </td>
   <td>Count
   </td>
   <td>Number of error packets (transmitted)
   </td>
  </tr>
  <tr>
   <td>tx_packets
   </td>
   <td>Count
   </td>
   <td>Number of total packets (transmitted)
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong>BlockIO - Scalable with "major" and "minor" id of devices</strong>
   </td>
  </tr>
  <tr>
   <td>io_merged_recursive
   </td>
   <td>Count
   </td>
   <td>Number of bios/requests merged into requests belonging to all the descendant cgroups
   </td>
  </tr>
  <tr>
   <td>io_queue_recursive
   </td>
   <td>Count
   </td>
   <td>Number of requests queued up at any given instant from all the descendant cgroups
   </td>
  </tr>
  <tr>
   <td>io_service_bytes_recursive
   </td>
   <td>Bytes
   </td>
   <td>Number of bytes transferred to/from the disk from all the descendant cgroups
   </td>
  </tr>
  <tr>
   <td>io_service_time_recursive
   </td>
   <td>Milliseconds
   </td>
   <td>Amount of time between request dispatch and request completion from all the descendant cgroups
   </td>
  </tr>
  <tr>
   <td>io_serviced_recursive
   </td>
   <td>Count
   </td>
   <td>Total number of block I/O requests serviced in that container
   </td>
  </tr>
  <tr>
   <td>io_time_recursive
   </td>
   <td>Milliseconds
   </td>
   <td>Disk time allocated to all devices from all the descendant cgroups
   </td>
  </tr>
  <tr>
   <td>io_wait_time_recursive
   </td>
   <td>Milliseconds
   </td>
   <td>Amount of time the IOs for this cgroup spent waiting in the scheduler queues for service from all the descendant cgroups
   </td>
  </tr>
  <tr>
   <td>sectors_recursive
   </td>
   <td>Count
   </td>
   <td>Number of sectors transferred to/from disk bys from all the descendant cgroups
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong>PIDs</strong>
   </td>
  </tr>
  <tr>
   <td>current
   </td>
   <td>Count
   </td>
   <td>Number of PIDs (Not available on Windows)
   </td>
  </tr>
</table>


### More about defining container filters  

In the **Container Filter** field, you can enter a comma-separated list of one or more of the following types of filters:

* A specific container name, for example, `my-container`
* A wildcard filter, for example, `my-container-*`
* An exclusion filter, which begins with an exclamation mark, for example, `!master-container` or `!prod-*`

For example, this filter list:

```
prod-*, !prod-*-mysql, master-*-app-*, sumologic-collector
```


will cause the source to collect from all containers whose names start with “prod-”, except those that match “prod-*-mysql”. It will also collect from containers with names that match “master-*-app-*”, and from the “sumologic-collector” container.

If your filter list contains only exclusions, the source will collect all containers except from those that match your exclusion filters. For example:


```
!container123*, !prod-*
```


will cause the source to exclude containers whose names begin with “container123” and “prod-”.


### Configure sourceCategory and sourceHost using variables

In collector version 19.216-22 and later, when you configure the sourceCategory and sourceHost for a Docker Log Source or a Docker Stats Source, you can specify the value using variables available from Docker and its host.

You build templates for sourceCategory and sourceHost specifying component variables in this form:


```
{{NAMESPACE.VAR_NAME}}
```


Where:

    •    `NAMESPACE` is a namespace that indicates the variable type. 

    •    `VAR_NAME` is the variable name.  These are case-sensitive.

The table below defines the types of variables you can use.


10
Docker engine events log data doesn't support the tagging with metadata.


<table>
  <tr>
   <td><strong>Namespace/VAR_TYPE</strong>
   </td>
   <td><strong>Description</strong>
   </td>
   <td><strong>VAR_NAME</strong>
   </td>
  </tr>
  <tr>
   <td><code>container</code>
   </td>
   <td>Container metadata fields provided by Docker for use in the <code>--log-opt</code> tag option.
<p>These are automatically added to data points.</p>
<p>For more information, see <a href="https://docs.docker.com/engine/admin/logging/log_tags/">Log tags for logging driver</a> in Docker help.</p>
   </td>
   <td><code>ID</code> The first 12 characters of the container ID.
<p><code>FullID</code> The full container ID.</p>
<p><code>Name</code> The container name.</p>
<p><code>ImageID</code> The first 12 characters of the container’s image ID.</p>
<p><code>ImageFullID</code> The container’s full image ID.</p>
<p><code>ImageName</code> The name of the image used by the container.</p>
   </td>
  </tr>
  <tr>
   <td><code>label</code>
   </td>
   <td>User-defined labels, supplied with the  <code>--label flag</code> when starting a Docker container.
<p>This is automatically added to data points.</p>
   </td>
   <td>The name of the variable.
<p>Dot characters (<code>.</code>) are not supported.</p>
   </td>
  </tr>
  <tr>
   <td><code>env</code>
   </td>
   <td>User-defined container environment variables that are set with <code>--env|-e</code> flags when starting a container.
   </td>
   <td>The name of the variable.
<p>Dot characters (<code>.</code>) are not supported.</p>
   </td>
  </tr>
</table>


For example:


```
{{container.ID}}
```


You can use multiple variables, for example:


```
{{container.ID}}-{{label.label_name}}-{{env.var_name}}
```


You can incorporate text in the metadata expression, for example:


```
ID{{container.ID}}-AnyTextYouWant{{label.label_name}}
```



11
The example above uses a hyphen `-` character to separate variable components. Separator characters are not required. Curly brackets and spaces are not allowed. Underscores and hyphens are recommended.

If a user-defined variable doesn’t exist, that portion of the metadata field will be blank.



## Install the Sumo Logic App

Now that you have set up collection of Docker logs and metrics, install the Sumo Logic App for Docker ULM to use the pre-configured searches and dashboards.

**To install the app**

Locate and install the app you need from the **App Catalog**. If you want to see a preview of the dashboards included with the app before installing, click **Preview Dashboards**.

1. From the **App Catalog**, search for and select the app**.**
2. To install the app, click **Add to Library**. \


    1. **App Name.** You can retain the existing name, or enter a name of your choice for the app. 
    2. **Docker Log Source.** Select the source category that you configured for the Docker Logs source. 
    3. **Docker Metrics Source**. Select the source category that you configured for the Docker Stats source.
    4. **Advanced**. Select the **Location in Library** (the default is the Personal folder in the library), or click **New Folder** to add a new folder.
    5. Click **Add to Library**.

Once an app is installed, it will appear in your **Personal** folder, or other folder that you specified. From here, you can share it with your organization. See [Welcome to the New Library](https://help.sumologic.com/01Start-Here/Welcome-to-the-New-Library) for information on working with the library .

Panels will start to fill automatically. It's important to note that each panel slowly fills with data matching the time range query and received since the panel was created. Results won't immediately be available, but with a bit of time, you'll see full graphs and maps.


## Viewing the Dashboards

This section describes the dashboards in the Sumo Logic App for Docker ULM.


#### Docker - Overview


See an overview of Docker activity, including the number of Docker hosts; the number of containers started, paused, stopped, and killed; event information; and the top containers by CPU usage, bytes sent, bytes received, and memory usage over the last 24 hours.


**Number of Docker Hosts**. Number of Docker hosts monitored over the last 24 hours.

**Number of Containers Started**.Number of containers started over the last 24 hours

**Number of Containers Paused**. Number of containers paused over the last 24 hours

**Number of Containers Stopped**. Number of containers stopped over the last 24 hours

**Number of Containers Killed**. Number of containers killed over the last 24 hours

**Docker Events Over Time**. A line chart that shows the count of specific Docker container events (such as pause, stop, die, restart, start, kill, unpause, and so on) per time slice over the last 24 hours.

**Containers by State**. A table that lists, for container events that occurred during the last 24 hours, the time the event occurred, the container image, the container name, and the event type.

**Container Events - One Day Time Comparison**. A table that compares the count of a particular event (such as pause, stop, die, restart, start, kill, unpause, and so on) for a image/container combination over the time range (24 hours by default), compared with 24 hours previous to the time range.

**Top 5 Containers by CPU Usage**. A line chart that shows the CPU usage by the five containers that have used the most CPU over the last 24 hours.

**Top 5 Containers by Tx Bytes**. A line chart that shows the bytes transmitted by top five containers that have transmitted the most bytes  over the last 24 hours.

**Top 5 Containers by Rx Bytes**. A line chart that shows the bytes received by the five containers that have received  the most bytes  over the last 24 hours.

**Top 5 Containers by Memory Usage**. A line chart that shows the memory usage by the top five containers that used the most memory over the last 24 hours.


#### Docker - Block IO
17


See the number of block I/O requests serviced by each container, and the number of bytes transferred to and from disk by each container over the last three hours.


18


**Total Number of Block I/O Requests Serviced**. A line chart that shows the number of block I/O requests serviced by each container over the last three hours.

**Number of Bytes Transferred to/from the Disk**. A line chart that shows the number of bytes written to and read by each container over the last three hours.


#### Docker - CPU Usage
19


See information about CPU usage and container throttling over the last three hours.


20


**Total CPU Consumed by Container in Kernel Mode**. A line chart that shows the CPU consumed in kernel mode by each container over the last three hours.

**CPU Usage by Image Name**. A line chart that shows the CPU consumed by container image name per timeslice over the last three hours.

**CPU Usage in Kernel Mode**. A line chart that shows the CPU consumed in kernel mode by each container per timeslice over the last three hours.

**CPU Usage in User Mode**. A line chart that shows the CPU consumed in user mode by each container per timeslice over the last three hours.

**Time for which container was throttled**. A line chart that shows the duration for which each container's CPU was throttled over the last three hours.

**Count of Periods with Throttling Active**. A line chart that shows how many times each container's CPU was throttled over the last three hours.


#### Docker - Memory Usage
21


See information about container memory limits, memory usage, and page faults over the last three hours.


22


**Number of Times Container Hit Memory Limit**. A  line chart that shows the number of times that each container reached its memory limit over the last three hours.

**Memory Limit by Container**. A line chart that shows the memory limit for each container over the last three hours.

**Percentage of Memory Used by Container**. A line chart that shows the percentage of memory used by each container over the last three hours.

**Current and Max Memory Usage**. A line chart that shows the current and maximum memory used by each container over the last three hours.

**Count of Page Faults by Container**. A line chart that shows the number of page faults for each container over the last three hours.

**Memory that Cannot be Reclaimed**. A line chart that shows the amount of memory that cannot be reclaimed for each container over the last three hours.

**RSS Memory by Container**. A line chart that shows the amount of RSS memory for each container over the last three hours.


#### Docker - Network Usage
23


See information about network usage over the last three hours.


24


**Average Rx Bytes by Container**. A line chart that shows the average number of bytes received per timeslice by each container over the last three hours.  

**Average Tx Bytes by Container**. A line chart that shows the average number of bytes transmitted per timeslice by each container over the last three hours.  

**Count of Rx Dropped Packets**. A line chart that shows the count of received packets dropped by each container per timeslice over the last three hours.

**Rate of Incoming Bytes**. A line chart that shows the rate at which bytes were received per timeslice by each container over the last three hours.  

**Rate of Outgoing Bytes**. A line chart that shows the rate at which bytes were transmitted per timeslice by each container over the last three hours.  

**Count of Rx Error Packets**. A line chart that shows the count of error packets received per timeslice by each container over the last three hours.   

**Average Rx Packets by Container**. A line chart that shows the average packets received per timeslice by each container over the last three hours.   

**Average Tx Packets by Container**. A line chart that shows the average packets transmitted per timeslice by each container over the last three hours.

**Count of Tx Dropped Packets**. A line chart that shows the count of packets dropped during transmission per timeslice by each container over the last three hours.

**Rate of Rx Packets**. A line chart that shows the rate at which packets were received per timeslice by each container over the last three hours.

**Rate of Tx Packets**. A line chart that shows the rate at which packets were transmitted per timeslice by each container over the last three hours.

**Count of Tx Error Packets**. A line chart that shows the count of error packets transmitted per timeslice by each container over the last three hours.   