---
title: "How to setup Falco and Wazuh integration"
date: "2025-05-10"
description: "Guide to setup k8s cluster monitoring with Falco and send alerts
to Wazuh"
tags: ["Falco", "Wazuh", "syslog"]
---

# How to setup Falco and Wazuh integration

## Overview

This guide describes how to integrate [Falco](https://falco.org/) with
[Wazuh](https://wazuh.com/) using `syslog` as a transport mechanism.
The logs are collected from a Kubernetes cluster where Falco is deployed via
Helm, forwarded to a Wazuh host over UDP, and processed by Wazuh decoders and
rules to generate alerts.

### Pipeline Diagram

```bash
+-----------------------------+
|      Kubernetes Cluster     |
| +-------------------------+ |
| |        Falco DaemonSet  | |
| |   +-------------------+ | |
| |   | Falcosidekick     | | |
| |   | - sends via UDP   | | |
| |   |   to syslog host  | | |
| |   +-------------------+ | |
| +-------------------------+ |
+-----------------------------+
              |
              v  (UDP:10514)
+-----------------------------+
|         Wazuh Host          |
| +-------------------------+ |
| |        rsyslogd         | |
| |  /var/log/falco.log     | |
| +-------------------------+ |
| +-------------------------+ |
| | Wazuh Logcollector      | |
| | - reads falco.log       | |
| | - decodes JSON          | |
| | - applies rules         | |
| +-------------------------+ |
|        |                    |
|        v                    |
|   Internal Elasticsearch    |
+-----------------------------+
```

## Prerequisites

* Working Kubernetes cluster
* Helm 3 installed
* rsyslog running on Wazuh host
* Wazuh manager with access to the log file and custom decoders/rules enabled
* UDP port 10514 open and reachable from Kubernetes nodes

---

## Step 1: Deploy Falco with Falcosidekick

Save the following to `values-falco.yml`:

```yaml
falcosidekick:
  enabled: true
  config:
    syslog:
      host: wazuh
      port: 10514
      protocol: udp
      minimumpriority: "info"
    slack:
      webhookurl: ""
  webui:
    enabled: true
    user: "admin:pass"
    service:
      port: 80
      targetPort: 2802
    ingress:
      enabled: true
      hosts:
        - host: falco-ui.example.com
          paths:
          - path: /
            pathType: Prefix
      annotations:
        kubernetes.io/ingress.class: alb
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/actions.ssl-redirect: >-
          {"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port":\n
          "443", "StatusCode": "HTTP_301"}}
        alb.ingress.kubernetes.io/group.name: stage-alb
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80},{"HTTPS":443}]'
        alb.ingress.kubernetes.io/load-balancer-name: stage-alb
        alb.ingress.kubernetes.io/target-type: ip
      tls:
        - secretName: falco-ui-tls
          hosts:
            - falco-ui.example.com
    redis:
      storageSize: "10Gi"
      storageClass: "gp2"

falco:
  jsonOutput: true
  priority: debug
  append_output:
    - extra_fields:
        - wazuh_integration: "falco"
        - cluster: "stage"
```

Install or upgrade with Helm:

```bash
helm upgrade --install falco falcosecurity/falco \
  --namespace falco --create-namespace \
  -f values-falco.yml
```

---

## Step 2: Configure rsyslog

Create or edit `/etc/rsyslog.d/50-falco.conf`:

```bash
cat <<EOF > /etc/rsyslog.d/50-falco.conf
# /etc/rsyslog.d/50-falco.conf

template(name="FalcoWithMetadata" type="string"
  string="%timestamp% %hostname% %programname%: json_data:%msg%\n")

if $programname == 'Falco' then {
    action(type="omfile" file="/var/log/falco.log" template="FalcoWithMetadata")
    stop
}
EOF
```

> **Note:** This template ensures that each log line begins with the syslog header,
> followed by a `json_data:` prefix which is used by the Wazuh decoder for reliable
> JSON extraction. Using `programname == 'Falco'` allows precise filtering.

Restart `rsyslog` to apply:

```bash
systemctl restart rsyslog
```

---

## Step 3: Setup logrotate

Add a logrotate policy in `/etc/logrotate.d/falco`:

```bash
/var/log/falco.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 0640 root root
    postrotate
        /bin/systemctl kill -s HUP rsyslog
    endscript
}
```

---

## Step 4: Wazuh Configuration

### Add log source

In `/var/ossec/etc/ossec.conf`:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/falco.log</location>
</localfile>
```

### Add decoders

Create or edit a decoder file, e.g. `/var/ossec/etc/decoders/falco_decoders.xml`:

```xml
<decoder name="falco">
  <program_name>^Falco$</program_name>
</decoder>

<decoder name="falcojson">
  <parent>falco</parent>
  <prematch>json_data: </prematch>
  <plugin_decoder offset="after_prematch">JSON_Decoder</plugin_decoder>
</decoder>
```

> **Note:** The `prematch` tag ensures that the decoder only processes log lines
> beginning with `json_data:`. The `plugin_decoder` handles the actual JSON
> parsing after this marker, allowing clean field extraction.

### Add rules

Create or edit `/var/ossec/etc/rules/falco_rules.xml`:

```xml
<group name="falco">
  <!-- Rule IDs in Wazuh should be in the 100000–120000 range, according to the documentation:
  https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html -->
  <rule id="109000" level="0" noalert="1">
    <decoded_as>falco</decoded_as>
    <description>Grouping for the Falco rules.</description>
  </rule>

  <rule id="109001" level="3">
    <if_sid>109000</if_sid>
    <field name="priority">Notice|Info|Debug</field>
    <description>Falco informational messages.</description>
  </rule>

  <rule id="109002" level="4">
    <if_sid>109000</if_sid>
    <field name="priority">Warning</field>
    <description>Falco warning messages.</description>
  </rule>

  <rule id="109003" level="6">
    <if_sid>109000</if_sid>
    <field name="priority">Error</field>
    <description>Falco error messages.</description>
  </rule>

  <rule id="109004" level="12">
    <if_sid>109000</if_sid>
    <field name="priority">Critical|Emergency|Alert</field>
    <description>Falco critical alerts.</description>
  </rule>

</group>
```

Restart the Wazuh manager:

```bash
systemctl restart wazuh-manager
```

---

## Step 5: Testing & Verification

1) Generate a Falco event (e.g. create a file in a restricted directory):

```bash
touch /bin/testfile
```

2) Verify it appears in `/var/log/falco.log`
3) Check Wazuh alerts via `/var/ossec/logs/alerts/alerts.json` or Wazuh UI

---

## Conclusion

This integration provides a clean and scalable way to forward Falco events to
Wazuh, allowing your SIEM to benefit from Falco’s kernel-level threat detection
capabilities inside Kubernetes. You can extend it further with custom rules,
Slack notifications, or other Falcosidekick outputs.
