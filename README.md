# prometheus-data-generator
Creates arbitrary prometheus metrics.

## Why use this?
When creating Grafana dashboards or Prometheus alerts, it is common to make
mistakes. You define a threshold that they have to meet, but when modified the
next time you may forget those thresholds.

Using this tool, you can create data with the format you want and
thus being able to base alerts and graphics on data that resemble reality.

To use this, you'll create a configuration file in which you will define a
metric name, description, type and labels and sequences of certain operations.

For example, you'll be able to create a alarm called `http_requests` with the
labels `{path=/login/, return_code=200}` which will be updated as you wish.

## Configuration
There's an example configuration file called `config.yml` in the root of the
repository. It has the next format:

``` yaml
config:
  - name: well_monitoring
    description: Well monitoring parameters.
    type: gauge
    labels: [well, parameter]
    sequence:
      - time: 5
        time_wait: 1
        values: 40-60
        operation: inc
        labels:
          well: 309
          parameter: intake_pressure
      - time: 5
        time_wait: 1
        values: 80-100
        operation: inc
        labels:
          well: 309
          parameter: intake_temperature
      - time: 5
        time_wait: 1
        values: 1-10
        operation: inc
        labels:
          well: 309
          parameter: vibration_x
      - time: 5
        time_wait: 1
        values: 1-10
        operation: inc
        labels:
          well: 309
          parameter: vibration_y
```

The generated metric will be like this:

``` text
# HELP well_monitoring Well monitoring parameters.
# TYPE well_monitoring gauge
well_monitoring{parameter="intake_pressure",well="309"} 45.0
well_monitoring{parameter="intake_temperature",well="309"} 86.0
well_monitoring{parameter="vibration_x",well="309"} 2.0
well_monitoring{parameter="vibration_y",well="309"} 3.0
```

### Supported keywords
- `name`: The ![metric
  name](https://prometheus.io/docs/instrumenting/writing_clientlibs/#metric-names).
  [**Type**: string] [**Required**]
- `description`: The description to be shown as
  ![HELP](https://prometheus.io/docs/instrumenting/writing_clientlibs/#metric-description-and-help).
  [**Type**: string] [**Required**]
- `type`: It should be one of the ![supported](###supported-metric-types) metric
  types.  [**Type**: string] [**Required**]
- `labels`: The labels that will be used with the metric. [**Type**: list of
  strings] [**Optional**]
- `sequence.time`: Number of seconds that the sequence will be running.
  [**Type**: int] [**Required**]
- `sequence.time_wait`: The interval of seconds between each operation will be
  performed. 1 second is a sane number. [**Type**: int] [**Required**]
- `sequence.value`: The value that the operation will apply. It must be a single
  value. You must choose between `value` and `values`. [**Type**: int] [**Optional**]
- `sequence.values`: The range of values that will randomly be choosed and the
  operation will apply. It must be two values separed by a dash. You must choose
  between `value` and `values`. [**Type**: string (int-int)] [**Optional**]
- `sequence.operation`: The operation that will be applied. It only will be used
  with the gauge type, and you can choose between `inc`, `dec` or `set`. [**Optional**]
- `sequence.labels`: The labels of the sequence. They must be used if `labels`
  are declared. [**Optional**]

### Supported metric types
The ones defined ![here](https://prometheus.io/docs/concepts/metric_types/).
- Counter
- Gauge
- Histogram
- Summary

## Manual use

```bash
git clone https://github.com/RBT-Production/prometheus-data-generator
virtualenv -p python3 venv
pip install -r requirements.txt
python prometheus_data_generator/main.py
curl localhost:9000/metrics/
```

## Use in docker

``` bash
wget https://raw.githubusercontent.com/RBT-Production/prometheus-data-generator/master/config.yml
docker run -ti -v `pwd`/config.yml:/config.yml -p 127.0.0.1:9000:9000 \
    alexperezpujol/prometheus-data-generator
curl localhost:9000/metrics/
```

## Use in kubernetes
There's some example manifests in the `kubernetes` directory. There's defined a
service, configmap, deployment (with
![configmap-reload](https://github.com/jimmidyson/configmap-reload) configured)
and a Service Monitor to be used with the ![prometheus
operator](https://github.com/coreos/prometheus-operator).

You may deploy the manifests:

``` bash
kubectl apply -f kubernetes/ -n monitoring
kubectl port-forward service/prometheus-data-generator 9000:9000
curl localhost:9000/metrics/
```

You can edit the configmap as you wish and the configmap-reload will
eventually reload the configuration without killing the pod.

## Generate prometheus alerts unit tests
TODO

## Tests
TODO
