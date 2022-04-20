# FIPS mode

## What is FIPS?

According to [Wikipedia](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards) 
> Federal Information Processing Standards (FIPS) are publicly announced standards developed by the National Institute of Standards and Technology for use in computer systems by non-military American government agencies and government contractors.
> FIPS standards are issued to establish requirements for various purposes such as ensuring computer security and interoperability and are intended for cases in which suitable industry standards do not already exist.

In short it is a thoroughly tested and verified set of standards which could be used to implement high level of security. In terms of software we are usually speaking specifically about FIPS 140-2. 

ReadonlyREST uses OpenSource [BouncyCastle](https://www.bouncycastle.org) library to provide FIPS 140-2 compliant algorithms.

## Does ReadonlyREST is fully FIPS compliant?

Unfortunately not yet. Currently only SSL transport part uses FIPS compliant algorithms.

## How to enable SSL FIPS compliance

1. Prepare keystore and truststore in BCFKS format which is FIPS compliant. It is described [in this section](#how-to-generate-bcfks-keystore-and-truststore-files). 
> :warning: BCFKS format is supported only when FIPS mode is enabled. It won't be recognised otherwise.

2. Configure readonlyrest.yml to use new keystore and truststore. You will also need to add new configuration parameter `fips_mode`. Here's an example:
``` 
readonlyrest:
  fips_mode: SSL_ONLY
  ssl:
    enable: true
    keystore_file: "keystore.bcfks"
    keystore_pass: readonlyrest
    truststore_file: "truststore.bcfks"
    truststore_pass: readonlyrest
    key_pass: readonlyrest

  ssl_internode:
    enable: true
    keystore_file: "keystore.bcfks"
    keystore_pass: readonlyrest
    key_pass: readonlyrest
    truststore_file: "truststore.bcfks"
    truststore_pass: readonlyrest
```
3. In case you are using ES >= 7.10 you need to modify `$JAVA_HOME/conf/security/java.policy` file and add this section at the end of it. It is required because ES restricted permissions that could be used by plugins.
```
grant {
  permission org.bouncycastle.crypto.CryptoServicesPermission "exportSecretKey";
  permission org.bouncycastle.crypto.CryptoServicesPermission "exportPrivateKey";
  permission java.security.SecurityPermission "getProperty.jdk.tls.disabledAlgorithms";
  permission java.security.SecurityPermission "getProperty.jdk.certpath.disabledAlgorithms";
  permission java.security.SecurityPermission "getProperty.keystore.type.compat";
  permission java.security.SecurityPermission "removeProvider.SunRsaSign";
  permission java.security.SecurityPermission "removeProvider.SunJSSE";
  permission java.io.FilePermission "${java.home}/lib/security/jssecacerts", "read";
  permission java.io.FilePermission "${java.home}/lib/security/cacerts", "read";
  permission java.security.SecurityPermission "getProperty.jdk.tls.server.defaultDHEParameters";
  permission org.bouncycastle.crypto.CryptoServicesPermission "defaultRandomConfig";
};
```


## How to generate BCFKS keystore and truststore files
