# ECK 

ReadonlyREST plugins officially support installation on Elasticsearch and Kibana working in K8S cluster and managed by the [ECK operator](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html).

## Configuration

### Elasticsearch node with ReadonlyREST plugin

If we want to add the ReadonlyREST plugin to the simple Elasticsearch cluster specification from [ECK Quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-elasticsearch.html) we should do it as below:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 8.14.3
  # check https://hub.docker.com/r/beshultd/elasticsearch-readonlyrest
  image: beshultd/elasticsearch-readonlyrest:8.14.3-ror-latest 
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
    podTemplate:
        spec:
          containers:
            - name: elasticsearch
              # we have to run our image as root (id: 0) - after the required patching step Elasticsearch will be run using "elasticsearch" user (id: 1000)
              securityContext:
                runAsNonRoot: false
                runAsUser: 0
                runAsGroup: 0
              env:
                # we have to explicitly agree to patch the ES binaries (the patching step will be done only once)
                - name: I_UNDERSTAND_IMPLICATION_OF_ES_PATCHING
                  value: "yes"
                # these two passwords are used by "elastic-internal" and "elastic-internal-probe" users - these users are used by ECK
                - name: INTERNAL_USR_PASS
                  valueFrom:
                    secretKeyRef:
                      name: quickstart-es-internal-users
                      key: elastic-internal
                - name: INTERNAL_PROBE_PASS
                  valueFrom:
                    secretKeyRef:
                      name: quickstart-es-internal-users
                      key: elastic-internal-probe
                # Kibana service account to handle internal Kibana requests 
                - name: KIBANA_SERVICE_ACCOUNT_TOKEN
                  valueFrom:
                    secretKeyRef:
                      name: quickstart-kibana-user
                      key: token
              # the initial readonlyrest.yml file loaded by ROR plugin during ES startup
              volumeMounts:
                - name: config-ror
                  mountPath: /usr/share/elasticsearch/config/readonlyrest.yml
                  subPath: readonlyrest.yml
          volumes:
            - name: config-ror
              configMap:
                name: config-readonlyrest.yml
```

The ReadonlyREST initial config can be defined as ConfigMap like this:

```yaml
apiVersion: v1
data:
   readonlyrest.yml: |
     readonlyrest:
       access_control_rules:

       - name: "ELASTIC-INTERNAL"
         verbosity: error
         auth_key: "elastic-internal:${INTERNAL_USR_PASS}"
     
       - name: "ELASTIC INTERNAL PROBE"
         verbosity: error
         auth_key: "elastic-internal-probe:${INTERNAL_PROBE_PASS}"
       
       - name: "Kibana service account"
         verbosity: error
         token_authentication:
           token: "Bearer ${KIBANA_SERVICE_ACCOUNT_TOKEN}" 
           username: service_account

       - name: "Admin access"
         type: allow
         auth_key: "admin:admin"

kind: ConfigMap
metadata:
  name: config-readonlyrest.yml
```

### Kibana node with ReadonlyREST plugin

If we want to add the ReadonlyREST plugin to the simple Kibana instance specification from [ECK Quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-kibana.html) we should do it as follows:

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 8.14.3
  # check https://hub.docker.com/r/beshultd/kibana-readonlyrest
  image: beshultd/kibana-readonlyrest:8.14.3-ror-latest 
  count: 1
  elasticsearchRef:
    name: quickstart
  config:
    # define ROR Kibana settings 
    # readonlyrest_kbn.store_sessions_in_index: true # we have to set it to true when we define more than one node
    readonlyrest_kbn.cookiePass: "12345678901234567890123456789012345678901234567890"
  podTemplate:
    spec:
      # we have to run our image as root (id: 0) - after the required patching step Kibana will be run using "kibana" user (id: 1000)
      securityContext:
        runAsNonRoot: false
        runAsUser: 0
        runAsGroup: 0
      containers:
        - name: kibana
          env:
            # we have to explicitly agree to patch the KBN binaries (the patching step will be done only once)
            - name: I_UNDERSTAND_IMPLICATION_OF_KBN_PATCHING
              value: "yes"
            # we have to provide a ROR license if we want to use ROR Pro or Enterprise
            - name: ROR_ACTIVATION_KEY
              value: "<YOUR_ACTIVATION_KEY/>"
```

And these are all differences we need to make to run [ECK Quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html) with ReadonlyREST instead of X-Pack security. 

## Notes

* [ReadonlyREST SSL](https://docs.readonlyrest.com/elasticsearch#encryption) can be used but it's simpler to leave `xpack.security.enabled: true` and use X-Pack SSL instead
* If running POD with root privileges is something you cannot accept, you can create your own image with the patching step done at the image creation level (not in the runtime as our image does) and place it your your registry
* It's possible, but we don't recommend installing the ReadonlyREST plugin using the init-container pattern, because of the patching required patching step