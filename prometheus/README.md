# MPP_Solar

You can also find these instructions on the top row within the dashboard.
This Dashboard takes avantage of using prom output from mpp_solar with command output from QPGS and QPIGS.
This output can be directed to a text file while building the file, then moved to node_exporter folder as a prom file.
This works with all *nix.

Requires a working installation of Prometheus_node_exporter on the same system with collection from folder enabled for node_exporter. Here is an example though I use an rc busybox based OS and your init system is likelly Systemd. The important part is to add a folder for node_exporter to read the prometheus formated files and enable node_exporter to do so ` --collector.textfile.directory=/var/lib/node-exporter/mppsolar` and your needs of where to add this switch will depend on your choice of distribution.

Designed originally for Rich Solar LV6548 and should work for MPP LV6548 and the like. Should also work with other inverters though thresholds should be adjusted for each case based on the models abilities.
I have created many panels to pick and choose from so you can remove what you do not want.
All panels are made as library items, so you can restore any panel that has been removed.

Some of the Array power guages, have a dynamic MAX set according to environment for each. For other inverter models, the multiplier value may need to be changed in the equation which can be adjusted in the dashboard variables found in the "Dashboard Settings". This is usually asked for during the dashboard import, though can be adjusted after.
The following status information needs to be poled for this dashboard:
```
QPGS 1-9 for each controller returns a bulk of the nedded metrics.
QPIRI for each controller sets some of the guages min and/or max values. (optional)
QPIGS for each controller returns invertter temperatures, solar pv1 power after MPPT. (optional)
QPIGS2 for each controlller returns solar pv2 (and maybe pv3 for controllers with 3rd input) power after MPPT. (optional)

```

Example with configurations broken out by commands:
```
# cat /etc/init.d/node-exporter-mpp
#!/sbin/openrc-run
supervisor=supervise-daemon

command="/usr/bin/node_exporter"
command_args="--no-collector.mdadm --collector.textfile.directory=/var/lib/node-exporter/mppsolar $ARGS"
command_background="yes"
group="prometheus"
user="prometheus"

logdir="/var/log/prometheus"
logfile="$logdir/${SVCNAME}.log"
datadir="/tmp/node_exporter"
pidfile="/var/run/${SVCNAME}.pid"
start_stop_daemon_args="--stderr $logfile --user $user --group $group"

depend() {
	need net
	after firewall
}

start_pre() {
	checkpath -d -o $user:$group -m755 $logdir
	checkpath -f -o $user:$group -m644 $logfile
        checkpath -f -o $user:$group -m755 $datadir
}
```


Designed for Rich Solar 6548 and should work for MPP LV6548 and the like. Should also work with other inverters though thresholds should be adjusted for each case based on the models abilities.

Adjust your serial ports as needed. 

`Example configurations:`
```
# cat /etc/mppsolar/mpp_qpgs.conf
# This pulls a bulk of the metrics used for graphing. (Required)
[SETUP]
pause=5

[Inverter_2]
port=/dev/ttyUSB2
protocol=PI30max
command=QPGS2
tag=QPGS2
outputs=prom
dev=SCC2

[Inverter_1]
port=/dev/ttyUSB0
protocol=PI30max
command=QPGS1
tag=QPGS1
outputs=prom
dev=SCC1
```
```
 # cat /etc/mppsolar/mpp_qpiri.conf
### This config retrives some of the specs for the controllers to set the gauges MIN/MAX values.  (optional)
[SETUP]
pause=5

[Inverter_2]
port=/dev/ttyUSB2
protocol=PI30max
command=QPIRI
outputs=prom
dev=SCC2

[Inverter_1]
port=/dev/ttyUSB0
protocol=PI30max
command=QPIRI
outputs=prom
dev=SCC1
```

```
 # cat /etc/mppsolar/mpp_qpigs.conf
# This output required for Inverter heatsink temp, and PV1 MPPT charge wattage. (optional)
[SETUP]
pause=5

[Inverter_2]
port=/dev/ttyUSB2
protocol=PI30max
command=QPIGS
outputs=prom
dev=SCC2

[Inverter_1]
port=/dev/ttyUSB0
protocol=PI30max
command=QPIGS
outputs=prom
dev=SCC1
```
```
# cat /etc/mppsolar/mpp_qpigs2.conf
### This config for inverters with more than one PV input, to retrive PV2 MPPT Charge Wattage output. (optional)
### Thoogh not a requirement with this dashboard.
[SETUP]
pause=5

[Inverter_2]
port=/dev/ttyUSB2
protocol=PI30max
command=QPIGS2
outputs=prom
dev=SCC2

[Inverter_1]
port=/dev/ttyUSB0
protocol=PI30max
command=QPIGS2
outputs=prom
dev=SCC1
```
Executable exammple in /usr/local/bin/
```
# cat /usr/local/bin/mpp-get-data.sh
#!/bin/ash
dataFile="mppsolar"
dataDir="/var/lib/node-exporter/mppsolar/"

checkpath -d "$dataDir" -m 755 -o prometheus:prometheus

while : ; do

        `/usr/bin/mpp-solar -C /etc/mppsolar/mpp_qpgs.conf > ${dataDir}${dataFile}.$$` && \
        `/usr/bin/mpp-solar -C /etc/mppsolar/mpp_qpiri.conf > ${dataDir}${dataFile}.$$` && \
        `/usr/bin/mpp-solar -C /etc/mppsolar/mpp_qpigs.conf >> ${dataDir}${dataFile}.$$` && \
        `/usr/bin/mpp-solar -C /etc/mppsolar/mpp_qpigs2.conf >> ${dataDir}${dataFile}.$$` && \
        mv ${dataDir}${dataFile}.$$ ${dataDir}${dataFile}.prom
        sleep 5
done
```

Create a SysV or systemd startup for the executable.

For markdown syntax help: [commonmark.org/help](https://commonmark.org/help/)
