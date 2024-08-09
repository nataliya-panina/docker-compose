# script - installation node-exporter and prometheus
```
#!/bin/bash
cd /home/$USER/prometheus
docker rm -f /node_exporter

docker run -d --net="host" --name node_exporter -v "/:/host:ro,rslave" \
quay.io/prometheus/node-exporter:latest --path.rootfs=/host

docker compose up -d
echo "paninang 5" | curl --data-binary @- http://localhost:9091/metrics/job/net>
ss -tlpn
```
