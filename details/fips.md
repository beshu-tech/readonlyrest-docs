# FIPS mode

## What is FIPS?

According to [Wikipedia](https://en.wikipedia.org/wiki/Federal\_Information\_Processing\_Standards)

> Federal Information Processing Standards (FIPS) are publicly announced standards developed by the National Institute of Standards and Technology for use in computer systems by non-military American government agencies and government contractors. FIPS standards are issued to establish requirements for various purposes such as ensuring computer security and interoperability and are intended for cases in which suitable industry standards do not already exist.

In short it is a thoroughly tested and verified set of standards which could be used to implement high level of security. In terms of software we are usually speaking specifically about FIPS 140-2.

ReadonlyREST uses OpenSource [BouncyCastle](https://www.bouncycastle.org) library to provide FIPS 140-2 compliant algorithms.

## Is ReadonlyREST fully compliant to FIPS 140-2?

At the moment, ReadonlyREST can be configured as FIPS compliant only from the "data in transit" standpoint. That is, the SSL encryption of the HTTP and transport interfaces. Other aspects remain to be covered:

* Making all cryptographic algorithms FIPS compliant.
* Enforcing more strict security policies across whole ROR plugin in FIPS mode.

## How to enable SSL FIPS compliance

1. Prepare keystore and truststore in BCFKS format which is FIPS compliant. Your existing JKS or PKCS12 keystore could be easily converted to BCFKS. Process is described [in this section](fips.md#how-to-convert-jkspkcs12-keystore-files-into-bcfks).

> :warning: BCFKS format is supported only when FIPS mode is enabled. It won't be recognised otherwise.

> :warning: When using FIPS mode using different password for specific keystore elements is not supported and `key_pass` configuration field is ignored.

1. Configure readonlyrest.yml to use new keystore and truststore. You will also need to add new configuration parameter `fips_mode`. Here's an example:

```
readonlyrest:
  fips_mode: SSL_ONLY
  ssl:
    enable: true
    keystore_file: "keystore.bcfks"
    keystore_pass: readonlyrest
    truststore_file: "truststore.bcfks"
    truststore_pass: readonlyrest

  ssl_internode:
    enable: true
    keystore_file: "keystore.bcfks"
    keystore_pass: readonlyrest
    truststore_file: "truststore.bcfks"
    truststore_pass: readonlyrest
```

1. In case you are using ES >= 7.10 you need to modify `$JAVA_HOME/conf/security/java.policy` file and add this section at the end of it. This is required because otherwise Elasticsearch will not be able grant to our plugin all these permissions at the JVM level.

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

## How to convert JKS/PKCS12 keystore files into BCFKS

1. Download the [jar with bc-fips](https://repo1.maven.org/maven2/org/bouncycastle/bc-fips/1.0.2.3/bc-fips-1.0.2.3.jar) library and place it preferably in the same directory where you store keystore files to convert.
2. Open your terminal and go to directory with the keystore to convert
3. Use keytool with following parameters to perform the conversion:

```
keytool \
-importkeystore \
-srckeystore SOURCE_KEYSTORE_FILENAME  \
-destkeystore DEST_KEYSTORE_FILENAME \
-srcstoretype SOURCE_KEYSTORE_TYPE \
-deststoretype DEST_KEYSTORE_TYPE \
-srcstorepass SOURCE_KEYSTORE_PASSWORD \
-deststorepass DEST_KEYSTORE_PASSWORD \
-providerpath ./bc-fips-1.0.2.1.jar \
-provider org.bouncycastle.jcajce.provider.BouncyCastleFipsProvider
```

where:

* SOURCE\_KEYSTORE\_FILENAME - filename of the keystore(or truststore) that you want to convert.
* DEST\_KEYSTORE\_FILENAME - name of the output file.
* SOURCE\_KEYSTORE\_TYPE - type of keystore to convert. Must be JKS or PKCS12.
* DEST\_KEYSTORE\_TYPE - type of output keystore. Must be BCFKS.
* SOURCE\_KEYSTORE\_PASSWORD - password protecting keystore to convert.
* DEST\_KEYSTORE\_PASSWORD - password protecting output file. If you saved the bc-fips jar in a different path, remember to run it using the appropriate path instead of `./bc-fips-1.0.2.1.jar`
