# AMDGPU monitoring for Telegraf

This is a small setup to monitor your AMD graphics card using `amdgpu_top` via Grafana using telegraf and InfluxDB on a debian based system.

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

## Testing the script

If you want to test the script you can do it with the -d flag and throw some JSON at the stdin:

```sh
amdgpu_top -J > sample.json
# cancel the command straight away with ctrl C
```

Check the JSON is valid with `jq`:

```sh
jq . sample.json
```

Feed it to the script, and it should output a line of ILP:

```sh
$ cat sample.json | /usr/local/bin/amdgpu_tlgfd -d
gpu_stats average_power_w=13,edge_critical_temperature_c=100,edge_emergency_temperature_c=105,edge_temperature_c=35,fclk_mhz=1591,fan_rpm=0,fan_max_rpm=3600,gfx_power_w=13,gfx_mclk_mhz=96,gfx_sclk_mhz=25,junction_critical_temperature_c=110,junction_emergency_temperature_c=115,junction_temperature_c=36,memory_critical_temperature_c=108,memory_emergency_temperature_c=113,memory_temperature_c=56,vddgfx_mv=635 1761217671000000000
```

Also if you simply run it, it will just continuously print polling amdgpu stats until you kill it.

```sh
$ /usr/local/bin/amdgpu_tlgfd
gpu_stats average_power_w=13,edge_critical_temperature_c=100,edge_emergency_temperature_c=105,edge_temperature_c=35,fclk_mhz=1591,fan_rpm=0,fan_max_rpm=3600,gfx_power_w=13,gfx_mclk_mhz=96,gfx_sclk_mhz=25,junction_critical_temperature_c=110,junction_emergency_temperature_c=115,junction_temperature_c=36,memory_critical_temperature_c=108,memory_emergency_temperature_c=113,memory_temperature_c=56,vddgfx_mv=635 1761217671000000000
gpu_stats average_power_w=13,edge_critical_temperature_c=100,edge_emergency_temperature_c=105,edge_temperature_c=35,fclk_mhz=1591,fan_rpm=0,fan_max_rpm=3600,gfx_power_w=13,gfx_mclk_mhz=96,gfx_sclk_mhz=31,junction_critical_temperature_c=110,junction_emergency_temperature_c=115,junction_temperature_c=37,memory_critical_temperature_c=108,memory_emergency_temperature_c=113,memory_temperature_c=56,vddgfx_mv=635 1761217672000000000
```

## Grafana setup

Setup your data source for influx in grafana, just make sure to set the server URL as `http://172.17.0.1:8086` and the database as `telegraf`.

<img width="686" height="613" alt="image" src="https://github.com/user-attachments/assets/070fa106-bb08-42a1-b8f6-8168089bb2fd" />


Then you wanna import the dashboard.json to grafana.

