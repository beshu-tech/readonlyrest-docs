# ECK 

ReadonlyREST plugins officially support installation on Elasticsearch and Kibana working in Kubernetes cluster and managed by the [ECK operator](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html).

You can choose one of the following installation methods:

## Installation methods 

### Using our Docker images from Docker Hub

The easiest and fastest method to start with a ROR-powered ELK stack on ECK is to use our official images.
We will show you how to do it in the following sections:

#### Elasticsearch node with ReadonlyREST plugin

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

##### ReadonlyREST initial settings

The initial settings can be defined as ConfigMap like this:

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

Notice that if you use ROR Enterprise, you can take advantage of the [Cluster-wide Settings](kibana.md#cluster-wide-settings-vs-readonlyrestyml) functionality and reload configuration on all your nodes without restarting K8s' PODs. 

#### Kibana node with ReadonlyREST plugin

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
            - name: I_UNDERSTAND_AND_ACCEPT_KBN_PATCHING
              value: "yes"
            # we have to provide a ROR license if we want to use ROR Pro or Enterprise (if the license is not provided, then ROR Free is used)
            - name: ROR_ACTIVATION_KEY
              value: "<YOUR_ACTIVATION_KEY/>"
```

And these are all differences we need to make to run [ECK Quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html) with ReadonlyREST instead of X-Pack security. 

### Using Docker images built by you and stored in your registry

As you probably noticed, our docker images have to be run with root privileges. It's due to legal reasons and you, as a user, 
have to confirm that you agree to do the patching steps. If running a pod with root privileges is something you cannot accept, you can create your own image with the patching step done at the image creation level (not at runtime as our image does) and save it your your own registry.

<details>
  <summary>Expand to see details</summary>
  
#### Elasticsearch with ROR custom image

The minimal Elasticsearch with ROR image definition looks like this:

```
# 'Dockerfile' file content
ARG ES_VERSION
FROM docker.elastic.co/elasticsearch/elasticsearch:${ES_VERSION}

ARG ES_VERSION
ARG ROR_VERSION

USER elasticsearch
RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch "https://api.beshu.tech/download/es?esVersion=$ES_VERSION&pluginVersion=$ROR_VERSION&email=[YOUR-EMAIL-ADDRESS]"
USER root
RUN /usr/share/elasticsearch/jdk/bin/java -jar /usr/share/elasticsearch/plugins/readonlyrest/ror-tools.jar patch
USER 1000:0
```

And then you can build it as follows:
```bash
docker build --build-arg ES_VERSION=8.14.3 --build-arg ROR_VERSION=1.59.0 -t elasticsearch-with-ror  .
```
And place the `elasticsearch-with-ror` image in your registry.

#### Elasticsearch node with ReadonlyREST plugin

If we want to add the ReadonlyREST plugin to the simple Elasticsearch cluster specification from [ECK Quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-elasticsearch.html) we should do it as below:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 8.14.3
  # this is the image from your registry
  image: elasticsearch-with-ror
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
    podTemplate:
        spec:
          containers:
            - name: elasticsearch
              env:
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

Check [the section from the previous paragraph](#readonlyrest-initial-settings) to see how to define `config-ror`.

#### Kibana with ROR custom image

The minimal Kibana with ROR image definition looks like this:

```
# 'Dockerfile' file content
ARG KBN_VERSION

FROM docker.elastic.co/kibana/kibana:${KBN_VERSION}

ARG KBN_VERSION
ARG ROR_VERSION

RUN /usr/share/kibana/bin/kibana-plugin install "https://api.beshu.tech/download/kbn?esVersion=$KBN_VERSION&pluginVersion=$ROR_VERSION&edition=kbn_universal&email=[YOUR-EMAIL-ADDRESS]"
USER root
RUN /usr/share/kibana/node/bin/node plugins/readonlyrestkbn/ror-tools.js patch && \
    chown -R kibana:kibana /usr/share/kibana/config
USER 1000:0
```

And then you can build it as follows:
```bash
docker build --build-arg KBN_VERSION=8.14.3 --build-arg ROR_VERSION=1.59.0 -t kibana-with-ror  .
```
And place the `kibana-with-ror` image in your registry.

#### Kibana node with ReadonlyREST plugin

If we want to add the ReadonlyREST plugin to the simple Kibana instance specification from [ECK Quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-kibana.html) we should do it as follows:

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 8.14.3
  # this is the image from your registry
  image: kibana-with-ror
  count: 1
  elasticsearchRef:
    name: quickstart
  config:
    # define ROR Kibana settings 
    # readonlyrest_kbn.store_sessions_in_index: true # we have to set it to true when we define more than one node
    readonlyrest_kbn.cookiePass: "12345678901234567890123456789012345678901234567890"
  podTemplate:
    spec:
      containers:
        - name: kibana
          env:
            # we have to provide a ROR license if we want to use ROR Pro or Enterprise (if the license is not provided, then ROR Free is used)
            - name: ROR_ACTIVATION_KEY
              value: "<YOUR_ACTIVATION_KEY/>"
```

And these are all differences we need to make to run [ECK Quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html) with ReadonlyREST instead of X-Pack security. 

</details>

### Using an Init Container 

{% hint style="warning" %}
This is not a recommended method.
{% endhint %}

It's possible to install ROR using an [Init Container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/). Unfortunately, this
method is not as easy to use in the case of ROR. This is due to the required ReadonlyREST patching step (for both plugins), which allows ROR to work with Elasticsearch/Kibana. During this step, the ROR patcher tool modifies some of the Elasticsearch and Kibana binaries. 
Different versions of Elasticsearch/Kibana require different binaries to be modified. In order to do this in a Kubernetes-based environment, 
we need to mount appropriate locations from the Elasticsearch/Kibana POD to the init container that will install ROR. The volumes should have write permissions because of the changes made by the ROR patcher. You can still use this method, but as you can see
it's not that easy.

<details>
  <summary>Expand to see details</summary>
  
If you are still interested in this one, please take a look at the examples in our repository:
* [Elasticsearch with ROR installed using the Init Container method](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/blob/v1.58.0_es8.14.3/docker-envs/eck/kind-cluster/ror/es.yml)
* [Kibana with ROR installed using the Init Container method](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/blob/v1.58.0_es8.14.3/docker-envs/eck/kind-cluster/ror/kbn.yml)

</details>

## Notes

* [ReadonlyREST SSL](https://docs.readonlyrest.com/elasticsearch#encryption) can be used but it's simpler to leave `xpack.security.enabled: true` and use X-Pack SSL instead
*  To figure out how to obtain the ROR License see [our guide](./universal-builds/universal-builds.md#how-to-activate-proenterprise-features-a-universal-build).

## Example

You can check the ROR-powered ECK Quickstart example by running our [one-liner script](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/tree/master/docker-envs/eck). It supports MacOS and Linux.