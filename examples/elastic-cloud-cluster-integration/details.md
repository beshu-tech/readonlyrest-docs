---
description: Detailed configuration
---

# Configuration details

This is a detailed description of how to configure two Elasticsearch clusters:

1. One in Elastic Cloud (managed Elasticsearch from Elastic) containing the bulk of the data
2. One self-hosted with ReadonlyREST (for enterprise-level access control and authentication)

The objective is to get the two connected using the transport protocol over SSL, so that we can attach a Kibana (with ROR Enterprise installed) to the cluster #2, and from there query the data in cluster #1 using the [Cross Cluster Search (CCS)](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html) feature.

## Two-way SSL configuration

The local, self-managed ROR cluster connects with the remote Elastic Cloud cluster using the Elasticsearch transport interface. The transport uses two-way SSL to authorize nodes of clusters.

To do that, we need to

1. Generate CA certificates of nodes of the local cluster (using the CA certificates of the Elastic cloud cluster)
2. Use them to add a trusted environment in the Elastic Cloud console
3. Configure the internode SSL and remote cluster settings in `elasticsearch.yml`

The CA certificates of the Elastic Cloud cluster nodes can be downloaded from the security settings of the Elastic Cloud deployment (see [screenshots](playgroud.md#running-interactive-script)).

### Generating ROR cluster CA and nodes' certificates

To generate CA certificates in the self-hosted cluster, we will use the `elasticsearch-certutil` which can be found in the `bin` folder in your Elasticsearch location (eg. `/usr/share/elasticsearch/bin/`).

Our working directory structure will look like that:

```bash
/tmp/certs# tree
.
|-- input
`-- output

2 directories, 0 files
```

Let's move the downloaded Elastic Cloud CA certificates file to `/tmp/certs/input` as `elastic-cloud-ca.cer`:

```bash
/tmp/certs# tree
.
|-- input
|   `-- elastic-cloud-ca.cer
`-- output

2 directories, 1 file
```

Now, let's create the `instances.yml` file in the `/tmp/certs/input` directory where we will define all nodes and their properties (see [Elastic instruction for details](https://www.elastic.co/guide/en/elasticsearch/reference/current/certutil.html#certutil-silent)) eg.

```yaml
instances:
  - name: "ror-es01" #{node name}
    cn:
      - "ror-es01.node.ror-cluster.ror-test" #{node name}.node.{cluster name}.{scope} (the scope will be useful during configuration of the trusted environments in Elastic Cloud deployment security settings)
    dns:
      - "localhost"
    ip:
      - "127.0.0.1"
```

Great, we have all the ingredients to generate the CA certificates of the nodes in our local ROR cluster:

```bash
mkdir -p /tmp/certs/output/ca
bin/elasticsearch-certutil ca --out /tmp/certs/output/ca/ca.p12 --pass mycapassword 
```

Details about the usage of the `elasticsearch-certutil` tool you will find in [Elastic documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/certutil.html). We have the CA certificate in `p12` format. We need to convert it to `X509`. It can be done using `openssl`:

```bash
openssl pkcs12 -in /tmp/certs/output/ca/ca.p12 -out /tmp/certs/output/ca/ca.crt -nokeys --password pass:mypassword 
```

Let's use our CA and generate certificates for the ROR cluster nodes:

```bash
bin/elasticsearch-certutil cert --silent --in /tmp/certs/input/instances.yml --out /tmp/certs/output/ror-cluster.zip --ca /tmp/certs/output/ca/ca.p12 --ca-pass mypassword --pass mypassword
unzip /tmp/certs/output/ror-cluster.zip -d /tmp/certs/output/ror-cluster
```

The last thing, we need to do, is to import Elastic Cloud CA to the ROR node's keystore:

```bash
jdk/bin/keytool -importcert -noprompt -file /tmp/certs/input/elastic-cloud-ca.cer -alias 'elastic-cloud' -keystore /tmp/certs/output/ror-cluster/ror-es01/ror-es01.p12 -storepass mypassword
```

This is it. The structure of the `certs` folder should look like this:

```bash
/usr/share/elasticsearch# tree /tmp/certs
/tmp/certs
|-- input
|   |-- elastic-cloud-ca.cer
|   `-- instances.yml
`-- output
    |-- ca
    |   |-- ca.crt
    |   `-- ca.p12
    |-- ror-cluster
    |   `-- ror-es01
    |       `-- ror-es01.p12
    `-- ror-cluster.zip

5 directories, 6 files
```

### Adding a new trusted environment in the Elastic Cloud deployment

In Elastic Cloud deployment security settings, there is a Remote Connections section, where you can add a new trusted environment (see [screenshots](playgroud.md#running-interactive-script)). The new trusted environment will be the self-managed cluster. To complete the process we need to:

1. upload the ROR cluster CA (`/tmp/certs/output/ca/ca.crt`)
2. select trusted cluster by:
   * ticking `Trust clusters whose Common Name follows the Elastic pattern`
   * entering `Scope ID` (in out example, it was `ror-test`)

* marking that we trust "All deployments" (or specific if you wish)

3. give a name of the environment (pick anything you want)
4. click `Create trust`

And that's it! Now ROR cluster should trust the Elastic Cloud cluster and vice versa.

## The minimal configuration of Elasticsearch & ReadonlyREST settings

`elasticsearch.yml` should look like this:

```yaml
cluster.name: ror-cluster # the same value used in `instances.yml`
node.name: ror-es01  # the same value used in `instances.yml`
network.host: 0.0.0.0

xpack.security.enabled: false

transport.type: ror_ssl_internode
readonlyrest: # we will put in in `elasticsearch.yml` because each node should have different certificate
  ssl_internode: 
    enable: true # we have to enable internode SSL because it's required to communicate with Elastic Cloud remote cluster
    keystore_file: "ror-cluster/ror-es01/ror-es01.p12"
    keystore_pass: "mypassword"
    truststore_file: "ror-cluster/ror-es01/ror-es01.p12"
    truststore_pass: "mypassword"
    key_pass: "mypassword"
    certificate_verification: true # it means that certificates will be validated
    client_authentication: true # ES with ROR acting as a client is going to authenticate itself

cluster.remote.escloud.mode: proxy # `escloud` is a remote cluster name - so to access `index1` on this remote cluster from the local cluster, we should refer it like that: `escloud:index1` (see `readonlyrest.yml` below) 
cluster.remote.escloud.proxy_address: '${ES_CLOUD_PROXY_ADDRESS}' # taken from Elastic Cloud deployment security settings, "Remote cluster parameters" section
cluster.remote.escloud.server_name: '${ES_CLOUD_SERVER_NAME}' # taken from Elastic Cloud deployment security settings, "Remote cluster parameters" section
```

and `readonlyrest.yml` like this:

```yaml
readonlyrest:

  access_control_rules:

    - name: "KIBANA" # for Kibana 
      type: allow
      auth_key: kibana:kibana

    - name: "ADMIN" # admin user - can change ROR settings
      type: allow
      kibana:
        access: admin
      auth_key: admin:admin
      
    - name: "User 1" # user1 can read remote Elastic Cloud cluster (escloud) indices matching pattern kibana_sample*
      type: allow
      kibana:
        access: ro
      auth_key: "user1:test"
      indices: [".kibana*", "escloud:kibana_sample*"]
```

Kibana configuration doesn't contain anything special.

<details>

<summary>Expand it if you really need to see how it looks like</summary>

\


`kibana.yml`:

```yaml
server.name: kibana-ror
server.host: 0.0.0.0
elasticsearch.hosts: [ "${ES_REST_API_URL}" ]
monitoring.ui.container.elasticsearch.enabled: true

elasticsearch.username: kibana
elasticsearch.password: kibana

# ReadonlyREST required properties
readonlyrest_kbn.cookiePass: '12312313123213123213123abcdefghijklm'
```

</details>
