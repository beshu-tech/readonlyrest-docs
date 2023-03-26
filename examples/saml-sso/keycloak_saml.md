---
description: SAML SSO Integration with Keycloak as an identity provider.
---

# Keycloak

This document will guide you through the task of setting up an excellent, open source identity provider ([KeyCloak](https://www.keycloak.org)) to work as an external authenticator and authorizer system for your ELK stack. The scenario is the usual:

* A centralised, large Elasticsearch cluster
* A Kibana installation
* We want one, centralised multi tenant Elasticsearch + Kibana

But with some more enterprise requirements:

* Users need to be able to change their passwords independently
* Users need to verify their emails
* Group managers need to be able to add, remove, block (only) their users.
* [Multi factor authentication (MFA)](https://www.keycloak.org/docs/latest/server\_admin/#one-time-password-otp-policies) is a requirement.

## What is Keycloak

Keycloak is an advanced authentication server that lets user administer their credentials, and speaks many authentication protocols, Including SAML2.0 SSO.

### Setup KeyCloak

This tutorial was created using KeyCloak 8.0.1.

1. Download the standalone version of Keycloak from their official website
2. Run Keycloak: run `bin/standalone.sh` or equivalent for your platform.
3. Navigate to [http://localhost:8080](http://localhost:8080) and configure the admin user's credentials **don't forget to fill the email address!**
4. Login as admin
5. Follow the explanation below, or (if your KC version is the same or close enough to this) use the import function to load this [configuration file](https://github.com/beshu-tech/readonlyrest-docs/tree/d77c4981b29a843fc82f89c4272fdddaab390d89/keycloak\_601\_ror\_SAML.json)

If you imported the JSON file, you should have a "ror" realm, and a SAML client called "ror" (keep this ID or change the "issuer" setting in kibana.yml) in the "master" realm. Please now select "ror" realm, navigate to "clients", click "ror" client and double check everything matches with your use case, as this guide assumes both Kibana, Elasticsearch and Keycloak are running on "localhost".

### Configure Keycloak to work with ROR

First we want to create a new dedicated "ror" realm, so we don't interfere with any other use of this Keycloak installation.

![keycloak\_screenshot](<../../.gitbook/assets/kc\_saml\_conf\_realm (1) (1) (1) (1) (1) (1) (1) (2) (5).png>)

Then, let's create a SAML client for this realm:

![keycloak\_screenshot](<../../.gitbook/assets/kc\_saml\_create\_saml\_client (1) (1).png>)

Then, configure the SAML client according to your Kibana URL, in this example, Kibana responds to "[https://localhost:5601/k](https://localhost:5601/k)"

![keycloak\_screenshot](<../../.gitbook/assets/kc\_saml\_configure\_saml\_client (1) (1).png>)

Now that the client is saved, let's observe the "configure" tab, here we will extract the two logout and login endpoints that we will use for configuring our SAML connector in "kibana.yml".

![keycloak\_screenshot](<../../.gitbook/assets/kc\_saml\_extract\_config\_from\_client (1) (1).png>)

### Install ReadonlyREST Enterprise for Kibana

Please refer to our [documentation](../../kibana.md) on how to obtain and install ReadonlyREST Enterprise for Kibana. Also remember that it relies on the Elasticsearch plugin to be configured as well.

### Setup the SAML connector

Provided that you have ReadonlyREST Enterprise installed configured, you can add the following configuration:

**kibana.yml**

```
# More on how to enable SSL on the official documentation of Kibana
server.ssl.enabled: true
server.ssl.key: /home/xx/selfsigned_ssl_localhost/localhost.key
server.ssl.certificate: /home/xx/selfsigned_ssl_localhost/localhost.crt

xpack.security.enabled: false

server.basePath: /k  # <-- optional, remember to change it in KC
elasticsearch:
  hosts: ["https://localhost:9200"] # <-- our Elasticsearch responds to https
  ssl.verificationMode: none
  username: kibana
  password: kibana

readonlyrest_kbn:
  logLevel: debug
  auth:
    # this secret string has to be longer than 256 chars, use environmental variables to fill it in maybe.
    signature_key: "9yzBfnLaTYLfGPzyKW9es76RKYhUVgmuv6ZtehaScj5msGpBpa5FWpwk295uJYaaffTFnQC5tsknh2AguVDaTrqCLfM5zCTqdE4UGNL73h28Bg4dPrvTAFQyygQqv4xfgnevBED6VZYdfjXAQLc8J8ywaHQQSmprZqYCWGE6sM3vzNUEWWB3kmGrEKa4sGbXhmXZCvL6NDnEJhXPDJAzu9BMQxn8CzVLqrx6BxDgPYF8gZCxtyxMckXwCaYXrxAGbjkYH69F4wYhuAdHSWgRAQCuWwYmWCA6g39j4VPge5pv962XYvxwJpvn23Y5KvNZ5S5c6crdG4f4gTCXnU36x92fKMQzsQV9K4phcuNvMWkpqVB6xMA5aPzUeHcGytD93dG8D52P5BxsgaJJE6QqDrk3Y2vyLw9ZEbJhPRJxbuBKVCBtVx26Ldd46dq5eyyzmNEyQGLrjQ4qd978VtG8TNT5rkn4ETJQEju5HfCBbjm3urGLFVqxhGVawecT4YM9Rry4EqXWkRJGTFQWQRnweUFbKNbVTC9NxcXEp6K5rSPEy9trb5UYLYhhMJ9fWSBMuenGRjNSJxeurMRCaxPpNppBLFnp8qW5ezfHgCBpEjkSNNzP4uXMZFAXmdUfJ8XQdPTWuYfdHYc5TZWnzrdq9wcfFQRDpDB2zX5Myu96krDt9vA7wNKfYwkSczA6qUQV66jA8nV4Cs38cDAKVBXnxz22ddAVrPv8ajpu7hgBtULMURjvLt94Nc5FDKw79CTTQxffWEj9BJCDCpQnTufmT8xenywwVJvtj49yv2MP2mGECrVDRmcGUAYBKR8G6ZnFAYDVC9UhY46FGWDcyVX3HKwgtHeb45Ww7dsW8JdMnZYctaEU585GZmqTJp2LcAWRcQPH25JewnPX8pjzVpJNcy7avfA2bcU86bfASvQBDUCrhjgRmK2ECR6vzPwTsYKRgFrDqb62FeMdrKgJ9vKs435T5ACN7MNtdRXHQ4fj5pNpUMDW26Wd7tt9bkBTqEGf"

    saml_kc:  # <--- Our SAML connector name, used in the path configured in KC
      buttonName: "KeyCloak SAML SSO"
      enabled: true
      type: "saml"
      issuer: "ror"  # <-- called exactly like the SAML client in KC
      entryPoint: "http://localhost:8080/auth/realms/ror/protocol/saml" # <-- from KC configuration tab!
      kibanaExternalHost: 'localhost:5601' 
      protocol: "https"  # <--- our Kibana responds to HTTPS
      usernameParameter: "nameID"
      groupsParameter: "Role"
      logoutUrl: "http://localhost:8080/auth/realms/ror/protocol/saml" # <-- from KC configuration tab!
      cert: /etc/ror/integration/certs/dag.crt # from KC realm keys tab <-- It can be also provided a string value 
```

You can find a public PEM-encoded X.509 signing certificate as a string value by selecting the "keys" tab in your newly created realm. After clicking on a cert button, you can copy the value into `kibana.yml` saml config `cert` parameter.

![keycloak\_screenshot](../../.gitbook/assets/kc\_saml\_keys\_tab.png)

Don't forget setting up SAML requires some changes to security settings in `readonlyrest.yml` (on the elasticsearch side). Security settings can also be changed via the ReadonlyREST Kibana app.

### Setup Elasticsearch with ReadonlyREST

Our Elasticsearch needs to be available on https (more detailed info in our [documentation](../../elasticsearch.md#encryption)), so we modify the elasticsearch.yml

**append to elasticsearch.yml**

```
xpack.security.enabled: false
http.type: ssl_netty4 # <-- needed for ROR SSL
```

Write in **readonlyrest.yml**

```
readonlyrest:

    ssl:
      enable: true
      keystore_file: "keystore.jks"
      keystore_pass: readonlyrest
      key_pass: readonlyrest

    prompt_for_basic_auth: false

    audit:
      enabled: true
      outputs:
      - type: index

    access_control_rules:
    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana
      verbosity: error

    - name: "ReadonlyREST Enterprise instance #1"
      kibana_index: ".kibana_sso"
      ror_kbn_auth:
        name: "kbn1"

    ror_kbn:
    - name: kbn1
      # It has to be the same string as we declared in kibana.yml.
      signature_key: "9yzBfnLaTYLfGPzyKW9es76RKYhUVgmuv6ZtehaScj5msGpBpa5FWpwk295uJYaaffTFnQC5tsknh2AguVDaTrqCLfM5zCTqdE4UGNL73h28Bg4dPrvTAFQyygQqv4xfgnevBED6VZYdfjXAQLc8J8ywaHQQSmprZqYCWGE6sM3vzNUEWWB3kmGrEKa4sGbXhmXZCvL6NDnEJhXPDJAzu9BMQxn8CzVLqrx6BxDgPYF8gZCxtyxMckXwCaYXrxAGbjkYH69F4wYhuAdHSWgRAQCuWwYmWCA6g39j4VPge5pv962XYvxwJpvn23Y5KvNZ5S5c6crdG4f4gTCXnU36x92fKMQzsQV9K4phcuNvMWkpqVB6xMA5aPzUeHcGytD93dG8D52P5BxsgaJJE6QqDrk3Y2vyLw9ZEbJhPRJxbuBKVCBtVx26Ldd46dq5eyyzmNEyQGLrjQ4qd978VtG8TNT5rkn4ETJQEju5HfCBbjm3urGLFVqxhGVawecT4YM9Rry4EqXWkRJGTFQWQRnweUFbKNbVTC9NxcXEp6K5rSPEy9trb5UYLYhhMJ9fWSBMuenGRjNSJxeurMRCaxPpNppBLFnp8qW5ezfHgCBpEjkSNNzP4uXMZFAXmdUfJ8XQdPTWuYfdHYc5TZWnzrdq9wcfFQRDpDB2zX5Myu96krDt9vA7wNKfYwkSczA6qUQV66jA8nV4Cs38cDAKVBXnxz22ddAVrPv8ajpu7hgBtULMURjvLt94Nc5FDKw79CTTQxffWEj9BJCDCpQnTufmT8xenywwVJvtj49yv2MP2mGECrVDRmcGUAYBKR8G6ZnFAYDVC9UhY46FGWDcyVX3HKwgtHeb45Ww7dsW8JdMnZYctaEU585GZmqTJp2LcAWRcQPH25JewnPX8pjzVpJNcy7avfA2bcU86bfASvQBDUCrhjgRmK2ECR6vzPwTsYKRgFrDqb62FeMdrKgJ9vKs435T5ACN7MNtdRXHQ4fj5pNpUMDW26Wd7tt9bkBTqEGf"
```
