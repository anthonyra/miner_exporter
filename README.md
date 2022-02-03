# miner\_exporter
Prometheus exporter for the [Helium miner (validator)](https://github.com/helium/miner). Using prometheus\_client, this code exposes metrics from the helium miner to a prometheus compatible server. 

This is only the exporter, which still requires a **prometheus server** for data and **grafana** for the dashboard. Prometheus and Grafana servers can run on an external machine, the same machine as the miner, or possibly using a cloud service. The [helium\_miner\_grafana\_dashboard](https://github.com/tedder/helium_miner_grafana_dashboard) can be imported to Grafana.

Note [port 9825 is the 'reserved' port for this specific exporter](https://github.com/prometheus/prometheus/wiki/Default-port-allocations). Feel free to use whatever you like, of course, but you won't be able to [dial 9VAL on your phone](https://en.wikipedia.org/wiki/E.161).


## Running via Docker
Using the docker file, you can run this with Docker or docker-compose! Both of these expose Prometheus on 9825, feel free to choose your own port. 

### Build docker image
Build the docker image and add it to your local image repository
```docker build -t miner_exporter:latest . ```

### Docker client
```
docker run -p 9825:9825 --name miner_exporter -v /var/run/docker.sock:/var/run/docker.sock miner_exporter:latest
```

### Docker-Compose
Using your existing docker-compose file, add the section for the exporter (below). When you're done, run `docker-compose up -d` as usual. That's it!
```
version: "3"
services:
  validator:
    image: quay.io/team-helium/validator:latest-val-amd64
    container_name: validator
...
  miner_exporter:
    image: miner_exporter:latest
    container_name: miner_exporter
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    ports:
    - "9825:9825"
    # Optional parameters
    environment:
    - UPDATE_PERIOD=25
```

## Running locally
On the miner machine:

install python3, python3-venv

```
pip install prometheus_client
```
Details on the libraries:
* [client\_python](https://github.com/prometheus/client_python)

Then install the service in a home directory:

```
sudo make install
```

Then install the systemd unit file:

```
sudo make install-service
```

Then enable and start the service:

```
sudo systemctl enable validator_exporter
sudo systemctl start validator_exporter
```

## Configuration

The following parameters can be used to modify configuration and behavior of the exporter:
```
UPDATE_PERIOD  # seconds between scrapes, int
VALIDATOR_JSONRPC_ADDRESS # address to call jsonrpc methods, default: http://localhost:4467
MINER_EXPORTER_PORT # port which miner_exporter listens on, default: 9825
COLLECT_SYSTEM_USAGE # boolean, default: False
ALL_HBBFT # collect HBBFT performance metrics on all validators, not just this validator, default: False
ALL_PENALTIES # collect penalty and heartbeat metrics on all stakes validators, default: False
```
