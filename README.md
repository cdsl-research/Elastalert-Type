# Elastalert-Type

## 1.0 Description
This repository provides a set of example configurations for ElastAlert2, demonstrating how different rule types can be used to detect various log patterns in Elasticsearch.  
It includes five YAML configuration files: `config.yaml`, `frequency.yaml`, `spike.yaml`, `flatline.yaml`, and `blacklist.yaml`.  
Each file serves as a simple example to show how ElastAlert2 can detect log anomalies such as high error rates, sudden spikes, missing logs, and unauthorized access attempts.  
The purpose of this repository is to help users understand and implement the basic rule types of ElastAlert2 for log monitoring and alerting.

## 2.0 Requirements

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

## 3.0 Setup

### Reference
For detailed documentation, refer to the official ElastAlert2 repository:  
[https://github.com/jertel/elastalert2](https://github.com/jertel/elastalert2)


### 3.1 Python Setup

1. Install the latest version of Python.  
   Example (for Ubuntu 22.04):

   ```
   c0a22173@elast:~$ sudo apt update
   c0a22173@elast:~$ sudo apt install python3 python3-pip python3-venv -y
   ```

2. Create and activate a virtual environment:

   ```
   c0a22173@elast:~$ python3 -m venv elast
   c0a22173@elast:~$ source elast/bin/activate
   (elast) c0a22173@elast:~$
   ```

3. Verify Python and pip versions:

   ```
   (elast) c0a22173@elast:~$ python3 --version
   Python 3.14.0
   (elast) c0a22173@elast:~$ pip3 --version
   pip 24.2 from /home/c0a22173/elastalert-env/lib/python3.14/site-packages/pip (python 3.14)
   ```


### 3.2 ElastAlert Setup

Follow these steps to install ElastAlert2 from the official GitHub repository.

```
c0a22173@elast:~$ git clone https://github.com/jertel/elastalert2.git
c0a22173@elast:~$ source elast/bin/activate
(elast) c0a22173@elast:~$ pip install "setuptools>=11.3"
Requirement already satisfied: setuptools>=11.3 in ./elast/lib/python3.12/site-packages (80.9.0)
(elast) c0a22173@elast:~$ ls
elast  elastalert2
(elast) c0a22173@elast:~$ cd elastalert2
(elast) c0a22173@elast:~/elastalert2$ ls
CHANGELOG.md     Dockerfile  examples  README.md             SECURITY.md  tests
chart            docs        LICENSE   requirements-dev.txt  setup.cfg
CONTRIBUTING.md  elastalert  Makefile  requirements.txt      setup.py
(elast) c0a22173@elast:~/elastalert2$ pip install "setuptools>=11.3"
(elast) c0a22173@elast:~/elastalert2$ python setup.py install
/home/c0a22173/elast/lib/python3.12/site-packages/setuptools/__init__.py:92: _DeprecatedInstaller: setuptools.installer and fetch_build_eggs are deprecated.
!!
        ********************************************************************************
        Requirements should be satisfied by a PEP 517 installer.
        If you are using pip, you can try `pip install --use-pep517`.

        By 2025-Oct-31, you need to update your project and remove deprecated calls
        or your builds will no longer be supported.
        ********************************************************************************
.
.
.
Installing elastalert script to /home/c0a22173/elast/bin
Installing elastalert-create-index script to /home/c0a22173/elast/bin
Installing elastalert-test-rule script to /home/c0a22173/elast/bin
(elast) c0a22173@elast:~$
```

**Explanation:**  
- The `git clone` command downloads the official ElastAlert2 source code.  
- A Python virtual environment is activated to isolate dependencies.  
- `setuptools` ensures that installation requirements are met.  
- `python setup.py install` installs ElastAlert2 locally inside the virtual environment.  
- Once completed, the executables `elastalert`, `elastalert-create-index`, and `elastalert-test-rule` are available inside the virtual environment’s `bin` directory.

   
### 3.3 Configuration Setup

After installation, configure the following YAML files as part of your ElastAlert setup.

| File | Description |
|------|--------------|
| `config.yaml` | Main configuration file for ElastAlert2, defining the Elasticsearch host, buffer time, and rule folder. |
| `frequency.yaml` | Detects high frequency of error logs within a short time frame. |
| `spike.yaml` | Detects a sudden increase in warning-level syslogs. |
| `flatline.yaml` | Detects absence of syslog activity within a certain time period. |
| `blacklist.yaml` | Detects authentication failures originating from blacklisted IP addresses. |

## 4.0 File Descriptions

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

## 5.0 Example of Execution
Elastalert Execution
<img width="940" height="303" alt="image" src="https://github.com/user-attachments/assets/d04cc4ac-c66a-4bcd-becc-c34735c5c368" />

Redmine Alert Example
<img width="1498" height="713" alt="image" src="https://github.com/user-attachments/assets/2fa8a074-bfcf-42a5-8dff-b21727cabf6b" />

## 6.0 Conclusion
This repository demonstrates how to implement different ElastAlert2 rule types for basic log monitoring and alerting.  
Each YAML file provides a clear example of how to configure specific rule behaviors, such as frequency, spike, flatline, and blacklist detection.  
These examples can serve as templates for developing more advanced alerting rules in larger monitoring environments.

