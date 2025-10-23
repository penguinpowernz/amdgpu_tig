# AMDGPU monitoring for Telegraf

This is a small setup to monitor your AMD graphics card using `amdgpu_top` via Grafana using telegraf and InfluxDB.

<img width="1595" height="860" alt="image" src="https://github.com/user-attachments/assets/d0c0ef3f-52ca-47cd-8b57-c9fb70f472b4" />


## Installation

Install docker:

```
apt-get install docker.io
sudo usermod -aG docker $USER
```

Run the containers:

```
docker run --restart=always -d --name influx -p 8086:8086 influxdb:1.12
docker run --restart=always -d --name grafana -p 3000:3000 grafana/grafana
```

Install telegraf:

```
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.36.3-1_amd64.deb
sudo dpkg -i telegraf_1.36.3-1_amd64.deb
```

Install ruby:

```
apt-get install --no-recommends ruby
```

Install `amdgpu_top`:

```
wget https://github.com/Umio-Yasuno/amdgpu_top/releases/download/v0.11.0/amdgpu-top_0.11.0-1_amd64.deb
sudo dpkg -i amdgpu-top_0.11.0-1_amd64.deb
```

Copy the files from this repo to the relevant places:
- telegraf configs
- systemd service
- amdgpu ruby script

Restart some things:

```
systemctl daemon-reload
systemctl restart telegraf
systemctl start amdgpu
```

## Grafana setup

Setup your data source for influx in grafana, just make sure to set the server URL as `http://172.17.0.1:8086` and the database as `telegraf`.

<img width="686" height="613" alt="image" src="https://github.com/user-attachments/assets/070fa106-bb08-42a1-b8f6-8168089bb2fd" />


Then you wanna import the dashboard.json to grafana.

