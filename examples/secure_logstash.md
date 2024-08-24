# Secure Logstash

We have a Logstash agent installed somewhere and we want to ship the logs to our Elasticsearch cluster securely.

## Elasticsearch side

**Step 1: Bring Elasticsearch HTTP interface \(port 9200\) to HTTPS** When you get SSL certificates \(i.e. from your IT department, or from LetsEncrypt\), you should obtain a private key and a certificate chain. In order to use them with ReadonlyREST, we need to wrap them into a JKS \(Java key store\) file. For the sake of this example, or for your testing, we won't use real SSL certificates, we are going to create a self signed certificate.

Remember, we'll do with a self-signed certificate for example convenience, but if you deploy this to a server, use a real one!

```bash
keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass readonlyrest -validity 360 -keysize 2048
```

Now copy the `keystore.jks` inside the plugin directory inside the Elasticsearch home.

```bash
cp keystore.jks /elasticsearch/config/
```

**IMPORTANT:** to enable ReadonlyREST's SSL stack, open `elasticsearch.yml` and append this one line:

```yaml
http.type: ssl_netty4
```

**Step 3** Now We need to create some credentials for logstash to login, let's say

* user = logstash
* password = logstash

**Step 4** Hash the credentials string `logstash:logstash` using SHA256. The simplest way is to paste the string in an [online tool](http://www.xorbin.com/tools/sha256-hash-calculator) You should have obtained "280ac6f756a64a80143447c980289e7e4c6918b92588c8095c7c3f049a13fbf9".

**Step 5** Let's add some configuration to our Elasticsearch: edit `conf/readonlyrest.yml` and append the following lines:

```yaml
 readonlyrest:

     ssl:
       enable: true
       # keystore in the same dir with readonlyrest.yml
       keystore_file: "keystore.jks"
       keystore_pass: readonlyrest
       key_pass: readonlyrest

     response_if_req_forbidden: Forbidden by ReadonlyREST ES plugin

     access_control_rules:

     - name: "::LOGSTASH::"
       auth_key_sha256: "280ac6f756a64a80143447c980289e7e4c6918b92588c8095c7c3f049a13fbf9" #logstash:logstash
       actions: ["cluster:monitor/main","indices:admin/types/exists","indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
       indices: ["logstash-*"]
```

## Logstash side

Edit the logstash configuration file and fix the output block as follows:

```ruby
 output {
   elasticsearch {
     ssl => true
     ssl_certificate_verification => false
     hosts => ["YOUR_ELASTICSEARCH_HOST:9200"]
     user => logstash
     password => logstash
   }
 }
```

The `ssl_certificate_verification` bit is necessary for accepting self-signed SSL certificates. You might also need to add cacert parameter to provide the path to your .cer or .pem file.
