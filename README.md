<!-- About Project -->
# 🚧 WORK IN PROGRESS 🚧
# Nvidia-SMI Telegraf Container
To aid in testing and experimenting with collecting statistics from an accelerated computing node with Nvidia GPUs. This is part of a project to show case visability of nodes visualized using a TIG stack. Refer to the repo [rjohnston6/ai-infra-monitoring][1] to setup a basic TIG stack to export data from each container deployed for monitoring multiple nodes.

<!-- Getting Started -->
## Getting Started
To simplify the deployment the use of a Compose is used to minimize the installation of software on the compute node. As a container runtime likely docker is typically found in a accelerated compute deployment limited installation is needed.

The container is tested on the following server configuration.

<!-- Add test configuration -->

### Requirements
- A container runtime of either Docker or Podman with the docker-compose utility installed.
- A InfluxDBv2 instance configured and running ready to accept export data

#### Step 1: Update Environment Variables
A number of variables are defined in the `telegraf.env` that need to be updated, the following should be updated to match your deployment.

| Variable | Default Value | Usage |
| :-- | :--: | :-- |
| INFLUXDB_URL | | Destination InfluxDBv2 IP/FQDN |
| VLLM_API_ADDRESS | | vLLM API IP/FQDN to poll token/second |
| INFLUXDB_ORG | ai-visibility | Destination InfluxDBv2 Organization |
| INFLUXDB_BUCKET | ai-visibility | Destination InfluxDBv2 Bucket |
| INFLUXDB_ADMIN_TOKEN | supersecuretoken | InfluxDBv2 administrator token to write data to bucket |

There are other variables defined that are used for allowing the container to probe the underlying hosts stats for CPU, these should not require modification.

#### Step 2: Update Telegraf.conf (Recommended)
Review the configuration of the `telegraf.conf` limited chnages are required to edit the hostname to assign to the container for easier reference when modeling data as well as the ports used by InfluxDBv2 instance and vLLM API default ports are assigned and if used with [rjohnston6/ai-infra-monitoring][1] should not require changes.

Locate the following snippits for any changes that may be required.

1. Updated Host Name
   ```ini
   ## Override default hostname, if empty use os.Hostname()
   hostname = "ai-11"
   ```
2. Update InfluxDBv2 listening if needed.
   ```ini
   [[outputs.influxdb_v2]]
     ## The URLs of the InfluxDB cluster nodes.
     ##
     ## Multiple URLs can be specified for a single cluster, only ONE of the
     ## urls will be written to each interval.
     ## urls exp: http://127.0.0.1:8086
     urls = ["http://${INFLUXDB_URL}:8086"]
   ```
3. Update vLLM Promothus Port if needed.
   ```ini
   [[inputs.prometheus]]
     urls = ["http://${VLLM_API_ADDRESS}:8000"]
     metric_version = 2

   [[inputs.http_response]]
     urls = ["http://${VLLM_API_ADDRESS}:8000/metrics"]
     response_timeout = "5s"
     method = "GET"
   ```

#### Step 3. Update Compose file
The compose file is configured and ready for deployment but depending on the number of GPUs configured updates to devices configuration is needed. The devices located under the `/dev/` directory. Using nvidia-smi and reviewing the `/dev/` directory map all available nvidia[0-7] devices.

An example snippet is below for 4 GPUs.
```yaml
devices:
  - /dev/nvidiactl:/dev/nvidiactl
  - /dev/nvidia0:/dev/nvidia0
  - /dev/nvidia1:/dev/nvidia1
  - /dev/nvidia2:/dev/nvidia2
  - /dev/nvidia3:/dev/nvidia3
```

By ensuring all GPUs are mapped statistics can be properly accessed from the container to the underlying host.

## Conclusion

<!-- Roadmap -->
## Roadmap
- [ ] 
- [ ] 
<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[1]: https://github.com/rjohnston6/ai-infra-monitoring

