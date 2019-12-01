# nomad-workload-cpu-actuals-report

## Summary

This is a tool that generates the historical workload output for nomad jobs and tasks, task groups based on metrics reported to prometheus. The output is an excel file with a worksheet for each environment sampled. Environments are polled concurrently to reduce the time it takes for the output to generate. The tool samples prometheus at the following intervals for each task:

- the past hour, sampling every 30 seconds
- the past day, sampling every 5 minutes
- the past 7 days, sampling every 30 minutes
- the past 30 days, sampling every hour

The query to prometheus is to/for the `avg_over_time` using the metric `nomad_client_allocs_cpu_total_percent`.

The tool reports out the following once per task group per environment:

1. Job Name
2. Task Group Name
3. Allocated MHz
4. Container Count

...and if enabled, for each 1 hour, 1 day, 7 days, 30 days:

1. Mean
2. 50th Percentile
3. 95th Percentile
4. Max
5. Standard Deviation
6. Kurtosis
7. Skewness

## Why use this?

If you've got a time series setup in place, you likely have grafana or other graphing tools in place to view the data and might ask what benefit a tool like this provides... Good question. This tool aims to help operators understand whether unbound (explain) workloads in Nomad are over or underschedule on machines **and** whether or not those workloads are stealing work from one another. It does so by essentially converting the load as a percentage of CPU used in sampling interval (prometheus) against the reported clockspeed of the Nomad client the workload was run on. It does this to produce "MHz used" such it can be compared to the CPU MHz reservation in the Nomad job definition.

## Prerequisites

1. Groovy installed (`brew install groovy`)
2. Nomad infrastructure with metrics reporting to prometheus
3. Keys to enable communication with your Nomad infrastructure
4. Unbound workloads in your Nomad container environment that need reviewing

## Usage

```
usage: ./nomadWorkloadCPUActualsReport -e <comma delimited environments> <other args>
Nomad Workload CPU Actuals Report time periods; 1h, 1d, 7d, 30d
 -avg,--avgKnownNomadClients           Use instead of --nomadFallbackClock to average the known Nomad clients for historical workloads
 -d1d,--disableOneDayQuery             Disable 1 day query
 -d1h,--disableOneHourQuery            Disable 1 hour query
 -d30d,--disableThirtyDayQuery         Disable 30 day query
 -d7d,--disableSevenDayQuery           Disable 7 day query
 -e,--environments <arg>               Environments (comma delimited)
 -fbc,--nomadFallbackClock <arg>       The approximate clockspeed of out-of-service Nomad nodes that were used to run historical workloads tracked in Prometheus [defaults to 2300]
 -h,--help                             Usage Information
 -nc,--nomadTLSCertFilename <arg>      Nomad TLS Key Filename [defaults to %env%.nomad.key]
 -nca,--nomadTLSCACertFilename <arg>   Nomad TLS CA Certificate Filename [defaults to nomadca.crt]
 -nh,--nomadHost <arg>                 Nomad host [defaults to https://nomad.service.%env%-dc1.consul:4646]
 -nk,--nomadTLSKeyFilename <arg>       Nomad TLS Certificate Filename [defaults to %env%.nomad.crt]
 -ph,--prometheusHost <arg>            Prometheus host [defaults to http://prometheus-read.service.%env%-dc1.consul:9090]
 -qst,--querySleepTime <arg>           Query sleep time in seconds [defaults to 0]
Environment queries are run in parallel to reduce report generation time. Use %env% to inject environment into --nomadTLSKeyFilename, --nomadTLSCertFilename, --nomadHost, --prometheusHost
```

## Understand the report

Reports are output with the timestamp from which the report generation process was started. Reports have a tab for each environment passed to the command line application. There are shaded or colored sections of the workbook that compare the following values to the `Nomad Job -> Task Group -> Tasks's` Allocated MHz:

1. Mean
2. 50th Percentile
3. 95th Percentile
4. Max

... with the colors:

- green: meaning a value is somewhere bewteen 0 and the Allocated MHz, richer meaning further (0 is darkest or richest green while closest to Allocated MHz is more green/white or white)
- yellow: meaning a value is somewhere between the Allocated MHz and 2x the Allocated MHz, richer meaning further (2x Allocated MHz is darkest or richest yellow, while closest to Allocated MHz is more yellow/white or white)
- red: meaning a value is or is greather than 2x the Allocated MHz

### See Also

- [Using Prometheus to Monitor Nomad Metrics](https://www.nomadproject.io/guides/operations/monitoring-and-alerting/prometheus-metrics.html)
- [Shape of data](https://brownmath.com/stat/shape.htm)
