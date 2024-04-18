# mppsolar_dashboards
Grafana dashboards, Prometheus rules, and associated files for use with mpp-solar by John Blance using a Prometheus or Victoria Metrics tdb backend.

I have created more panels than really necessary so those not comfortable with promql or grafana can move, size, and remove panels as they see fit.

All panels are part of a library, so any removed panel can be restored, or easily placed in another dashboard.

![Grafana dashboard](https://github.com/cordelster/mppsolar_dashboards/blob/main/prometheus/pics/dash_guages.png)

## New! Consolidated some of the status into a table..
![Grafana dashboard stats in table](https://github.com/cordelster/mppsolar_dashboards/blob/main/prometheus/pics/dash_tables.png)

Dashboards have some dynamic programed Total gauges that adjust with controller configuration. 
Total gauge for "Array power by inverter" automaticly adjusts it maximum value by how many inverters report back voltage on each array input.
Total gauge for "Battery charge current" adjusts its minimum and maximum capacity by by how many controllers report in use.
Total gauge for Total "AC power watts" output adjusts its maximum by how many inverters report is_on. This allows for adding inverters/controllers using the  same model without the need to adjust the dashboard.

TODO:
- Create variables in Dashboard to adjust values depending on inverter. Reduce the need to adjust equations directly.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/L3L0V38OP)
