---
author: spooky
pubDatetime: 2024-05-04T11:30:08Z
title: K6 operator on Kubernetes
featured: true
draft: false
tags:
  - how-to
  - load-testing
  - k6
canonicalURL: https://spookyintheam.com/posts/k6-operator-on-kubernetes/
description: How to use setup and use K6 operator on Kubernetes
---

The k6 operator can be installed on a Kubernetes cluster to enable load testing on the cluster and cloud in a distributed manner.

### Writing and saving tests

- Tests can be saved either using configMaps but there is a limit on the data that can be saved inside a configMap.
- We can use PVC and PV to store tests and then reference the volumeClaim to run these tests files.
  ```yaml
  apiVersion: k6.io/v1alpha1
  kind: TestRun
  metadata:
    name: run-k6-from-volume
  spec:
    parallelism: 4
    script:
      volumeClaim:
        name: my-volume-claim
        # File is relative to /test/ directory within volume
        file: test.js
  ```

### Getting values from env vars

- We can configure tests scripts to get values from env vars (system vars, CMs and k8s secrets) then refer to these in the script.

```yaml
apiVersion: k6.io/v1alpha1
kind: TestRun
metadata:
  name: k6-test
spec:
  parallelism: 4
  script:
    configMap:
      name: test-k6-cm
      file: test.js
  runner:
    envFrom:
      - secretRef:
          name: example-secret
```

- Then to refer to these:
  ```jsx
  export function setup() {
    // This variable is taken from the secret example-secret
    console.log(`Variable is set as: ${__ENV.username}`);
  }
  ```

### Prometheus and exporting to Grafana

```yaml
apiVersion: k6.io/v1alpha1
kind: TestRun
metadata:
  name: k6-test
spec:
	arguments: -o experimental-prometheus-rw
  parallelism: 4
  script:
    configMap:
      name: test-k6-cm
      file: test.js
  runner:
		env:
			- name: K6_PROMETHEUS_RW_SERVER_URL
				value: http://localhost:9090/api/v1/write
			- name: K6_PROMETHEUS_RW_TREND_STATS
				value: p(95),p(99),min,max,avg,count,sum
    envFrom:
      - secretRef:
          name: example-secret

```
