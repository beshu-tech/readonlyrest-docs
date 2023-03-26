---
description: OpenID Connect (OIDC) SSO Integration with Keycloak as an identity provider.
---

# Keycloak

This document will guide you through the task of setting up an excellent, open-source identity provider ([KeyCloak](https://www.keycloak.org)) to work as an external authenticator and authorizer system for your ELK stack. The scenario is the usual:

* A centralised, large Elasticsearch cluster
* A Kibana installation
* We want one, centralised multi tenant Elasticsearch + Kibana;

But with some more enterprise requirements:

* Users need to be able to change their passwords independently
* Users need to verify their emails
* Group managers need to be able to add, remove, block (only) their users.
* [Multi factor authentication (MFA)](https://www.keycloak.org/docs/latest/server\_admin/#one-time-password-otp-policies) is a requirement.

## What is Keycloak

Keycloak is an advanced authentication server that lets user administer their credentials, and speaks many authentication protocols, Including OpenID Connect (OIDC) SSO.

### Setup KeyCloak

This tutorial was created using KeyCloak 14.0.0.

1. Download the Keycloak from their [official website](https://www.keycloak.org/archive/downloads-14.0.0.html). This guide will use [keycloak docker image](https://hub.docker.com/r/jboss/keycloak/)
2. Run Keycloak: run docker run -e KEYCLOAK\_USER= -e KEYCLOAK\_PASSWORD= jboss/keycloak where USERNAME and PASSWORD are credentials for your admin account
3. log in as admin
4. Follow the explanation below, or (if your KC version is the same or close enough to this) use the import function to load this [configuration file](../../keycloak\_ror\_OIDC.json)

If you imported the JSON file, you should have a "ror" realm, and an OpenID Connect (OIDC) client called "ror\_oidc" (keep this ID or change the "clientID" setting in kibana.yml). Please now select "ror" realm, navigate to "clients", click "ror\_oidc" client and double-check everything matches with your use case, as this guide assumes both Kibana, Elasticsearch, and Keycloak are running on "localhost".

### Configure Keycloak to work with ROR

First, we want to create a new dedicated "ror" realm, so we don't interfere with any other use of this Keycloak installation.

![keycloak\_screenshot](<../../.gitbook/assets/kc\_saml\_conf\_realm (1) (1) (1) (1) (1) (1) (1) (2) (5).png>)

Then, let's create an OpenId Connect client for this realm:

![keycloak\_screenshot](../../.gitbook/assets/oidc\_create\_client.png)

Then, configure the OpenID Connect (OIDC) client

![keycloak\_screenshot](../../.gitbook/assets/oidc\_configure\_client.png)

**kibana.yml** (without ssl enabled)

```
# More on how to enable SSL on the official documentation of Kibana
server.ssl.enabled: false

xpack.security.enabled: false

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
    oidc_kc:
      buttonName: "KeyCloak OpenID"
      type: "oidc"
      protocol: "http"
      issuer: 'http://localhost:8080/auth/realms/ror' <-- Get it from OpenID Endpoint Configuration
      authorizationURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/auth' <-- Value from from OpenID Endpoint Configuration
      tokenURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/token' <-- Value from from OpenID Endpoint Configuration
      userInfoURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/userinfo' <-- Value from from OpenID Endpoint Configuration
      clientID: 'ror_oidc' <-- Declared in a realm Client Scopes
      clientSecret: '35d0c1db-a2b7-42d9-9a43-bea88c6535e6'  <-- Declared in a realm ror_oidc (our created client) Credentials tab
      scope: 'openid profile roles role_list email' <-- Declared in a realm Client Scopes
      usernameParameter: 'preferred_username'
      groupsParameter: 'groups'
      kibanaExternalHost: 'localhost:5601'
      logoutUrl: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/logout' <-- Value from from OpenID Endpoint Configuration
      jwksURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/certs' <-- Value from from OpenID Endpoint Configuration
```

To verify all OpenID Endpoint Configuration-based, you can open OpenID Endpoint Configuration page in the kibana realm

![keycloak\_screenshot](../../.gitbook/assets/oidc\_openid\_endpoint\_configuration.png)

To provide clientSecret value, you need to open ror\_oidc client (or your custom client name)

![keycloak\_screenshot](../../.gitbook/assets/oidc\_client\_secret.png)

### Setup Elasticsearch with ReadonlyREST

Our elasticsearch can be run with or without SSL. To make it available on HTTPS (more detailed info in our [documentation](../../elasticsearch.md#encryption)), so we modify the elasticsearch.yml

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
