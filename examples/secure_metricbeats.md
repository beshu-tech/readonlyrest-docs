# Secure Metricbeats

Very similarly to Logstash, here's a snippet of configuration for [Metricbeats](https://www.elastic.co/downloads/beats/metricbeat) logging agent configuration of metricbeat - elasticsearch section

## On the Metricbeats side

```text
output.elasticsearch:
  output.elasticsearch:
  username: metricbeat
  password: hereyourpasswordformetricbeat
  protocol: https
  hosts: ["xx.xx.xx.xx:9200"]
  worker: 1
  index: "log_metricbeat-%{+yyyy.MM}"
  template.enabled: false
  template.versions.2x.enabled: false
  ssl.enabled: true
  ssl.certificate_authorities: ["./certs/your-rootca_cert.pem"]
  ssl.certificate: "./certs/your_srv_cert.pem"
  ssl.key: "./certs/your_srv_key.pem"
```

Of course, if you do not use SSL, disable it.

## On the Elasticsearch side

```yaml
 readonlyrest:
     ssl:
       enable: true
       # keystore in the same dir with elasticsearch.yml
       keystore_file: "keystore.jks"
       keystore_pass: readonlyrest
       key_pass: readonlyrest

    access_control_rules:
    - name: "metricbeat can write and create its own indices"
      auth_key_sha1: fd2e44724a234234454324253094080986e8fda
      actions: ["indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
      indices: ["metricbeat-*", "log_metricbeat*"]
```
