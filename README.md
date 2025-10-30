# Elastalert-Type

## Brief Description
This repository provides a set of example configurations for ElastAlert2, demonstrating how different rule types can be used to detect various log patterns in Elasticsearch.  
It includes five YAML configuration files: `config.yaml`, `frequency.yaml`, `spike.yaml`, `flatline.yaml`, and `blacklist.yaml`.  
Each file serves as a simple example to show how ElastAlert2 can detect log anomalies such as high error rates, sudden spikes, missing logs, and unauthorized access attempts.  
The purpose of this repository is to help users understand and implement the basic rule types of ElastAlert2 for log monitoring and alerting.

---

## Requirements

| Component | Version |
|------------|----------|
| Nodes | 1 |
| Elasticsearch | 8.13.4 |
| ElastAlert2 | 2.11.1 |

| Resource | Specification |
|-----------|----------------|
| CPU | 4 cores |
| Memory | 8 GB |
| Storage | 40 GB |

---

## Contents

| File | Description |
|------|--------------|
| `config.yaml` | Main configuration file for ElastAlert2, defining the Elasticsearch host, buffer time, and rule folder. |
| `frequency.yaml` | Detects high frequency of error logs within a short time frame. |
| `spike.yaml` | Detects a sudden increase in warning-level syslogs. |
| `flatline.yaml` | Detects absence of syslog activity within a certain time period. |
| `blacklist.yaml` | Detects authentication failures originating from blacklisted IP addresses. |

---

## File Descriptions

### config.yaml
The `config.yaml` file defines how ElastAlert connects to Elasticsearch and how often it queries for logs.  
The key parameters are shown below:

```yaml
rules_folder: rules1
run_every:
  minutes: 1
buffer_time:
  minutes: 2
es_host: ls-master.a910.tak-cslab.org
es_port: 30092
```

**Explanation:**  
- `rules_folder` specifies the folder that contains the rule YAML files.  
- `run_every` determines how often ElastAlert queries Elasticsearch.  
- `buffer_time` defines how far back ElastAlert looks when querying logs.  
- `es_host` and `es_port` specify the Elasticsearch endpoint where logs are stored.  

This configuration ensures that ElastAlert runs every minute and queries the last two minutes of log data to detect events in near real-time.

---

### frequency.yaml
This rule detects when too many error messages appear in a short time frame within the `alert` namespace.

```yaml
type: frequency
index: "beats-*"
num_events: 10
timeframe:
  minutes: 10
filter:
  - query:
      query_string:
        query: 'kubernetes.namespace: alert AND message: error'
```

**Explanation:**  
- `type: frequency` triggers an alert when the number of matched events exceeds a specified limit.  
- `num_events: 10` defines the threshold for how many error logs must occur.  
- `timeframe: 10 minutes` limits the evaluation period.  
- The `filter` section defines the Elasticsearch query used to match logs with `message: error` inside the `alert` namespace.  
This rule is useful for detecting recurring errors in a short time window.

---

### spike.yaml
This rule detects a sudden spike in the number of warning logs in the `syslog-*` index.

```yaml
type: spike
index: "syslog-*"
timeframe:
  minutes: 30
spike_height: 3
spike_type: up
filter:
  - query:
      query_string:
        query: 'log.syslog.severity.name.keyword : "Warning"'
```

**Explanation:**  
- `type: spike` compares two consecutive time windows of the same length.  
- `spike_height: 3` means the log count must increase by a factor of three to trigger an alert.  
- `spike_type: up` specifies detection of upward spikes.  
- The `filter` section captures logs with severity level “Warning.”  
This rule is effective for identifying sudden bursts of warning logs, which may indicate new or growing issues in the system.

---

### flatline.yaml
This rule detects when no syslog entries are received within a specified period.

```yaml
type: flatline
index: "syslog-*"
threshold: 1
timeframe:
  minutes: 30
filter:
  - query:
      query_string:
        query: '*'
```

**Explanation:**  
- `type: flatline` triggers an alert when the number of events falls below the defined threshold.  
- `threshold: 1` means the alert triggers if fewer than one log is received during the timeframe.  
- The query `*` matches all logs in the index.  
This rule is used to detect logging interruptions, such as when a syslog source stops sending data.

---

### blacklist.yaml
This rule detects authentication failures originating from specific blacklisted IP addresses.

```yaml
type: blacklist
index: "beats-*"
filter:
  - query:
      query_string:
        query: 'source.ip: ("10.10.10.10" OR "10.10.10.11") AND (event.action: "authentication_failure" OR message: "failed password")'
```

**Explanation:**  
- `type: blacklist` triggers when an event matches a pattern associated with blocked or unauthorized sources.  
- The query filters logs that come from blacklisted IP addresses and include messages indicating authentication failures.  
This rule is effective for detecting repeated login attempts or suspicious activities from known bad IPs.

---

## Example of Execution
Elastalert Execution
<img width="942" height="285" alt="image" src="https://github.com/user-attachments/assets/a23dc1fd-f885-4956-a1d8-3dca2e5352b1" />
Redmine Alert Example
<img width="1498" height="713" alt="image" src="https://github.com/user-attachments/assets/2fa8a074-bfcf-42a5-8dff-b21727cabf6b" />

---

## Conclusion
This repository demonstrates how to implement different ElastAlert2 rule types for basic log monitoring and alerting.  
Each YAML file provides a clear example of how to configure specific rule behaviors, such as frequency, spike, flatline, and blacklist detection.  
These examples can serve as templates for developing more advanced alerting rules in larger monitoring environments.

