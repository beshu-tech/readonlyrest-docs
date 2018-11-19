## Integrating ReadonlyRest SAML with Duo Security

### Duo Security Configuration
#### Installing Duo Gateway
Follow the instructions for Duo Gateway (https://duo.com/docs/dag-linux). The gateway is a Docker container and it will need the ports 80 and 443 to be available, so you will probably need a dedicated VM or Bare Metal for Duo Gateway.

#### Configuring Authentication Source
When you configure the authentication sources, be sure to set the correct username attribute. This will be mapped in Duo Gateway > Applications.

![Source](clusterconfig)

#### Configure Application in Duo Admin Dashboard
In the Duo Administration dashboard, go to Applications. Click  `Protect an Application`, then search for `Generic` and click `Protect this Application` button.

This will create a generic SAML provider. Set these fields:
- Service provider name: A name that refers to your ReadonlyREST Enterprise  installation
- EntityID: Set an existing entity name or use the same as Service Provider Name
- Assertion Consumer Service: The SAML Url assertion found in metadata.xml, the url format is `<kibanaExternalHost>/ror_kbn_sso/assert`. 

![SP](sp)

In SAML Response, set NameID to be the same variable name as configured previously in the authentication source. Leave the default values for the remaining settings. Click `Save Application` and scroll up and download the configuration file.


### ReadonlyRest Configuration
To  configure SAML, both Kibana and Elasticsearch ROR configuration needs to be edited to enable SAML in Duo Gateway.
#### Elasticsearch
ror_kbn_auth
For Enterprise customers only, required for SAML authentication.

```yml
readonlyrest:
  access_control_rules:

    - name: "ReadonlyREST Enterprise instance #1"
      ror_kbn_auth:
        name: "kbn1"

    - name: "ReadonlyREST Enterprise instance #2"
      ror_kbn_auth:
        name: "kbn2"

  ror_kbn:
    - name: kbn1
      signature_key: "shared_secret_kibana1" # <- use environmental variables for better security!

    - name: kbn2
      signature_key: "shared_secret_kibana2" # <- use environmental variables for better security!
```
This authentication and authorization connector represents the secure channel (based on JWT tokens) of signed messages necessary for our Enterprise Kibana plugin to securely pass back to ES the username and groups information coming from browser-driven authentication protocols like SAML


#### Kibana
Edit $KIBANA_HOME/conf/kibana.yml configuration and append:
```yml
readonlyrest_kbn.auth:
  signature_key: “a very long key (more than 256 characters) goes here …..” # the same signing key must be added in es config
  saml:
    enabled: true
    entryPoint: 'https://duo-gateway.xyz/dag/saml2/idp/SSOService.php?spentityid=demo'
    kibanaExternalHost: 'ror-deployment.xyz' # <-- public URL used by the Identity Provider to call back Kibana with the "assertion" message
    usernameParameter: 'nameID'
    groupsParameter: 'memberOf'
    logoutUrl: 'https://duo-gateway.xyz/dag/saml2/idp/SingleLogoutService.php?ReturnTo=https://duo-gateway.xyz/dag/module.php/duosecurity/logout.php'
    decryptionCert: certs/dag.crt
    cert: certs/dag.crt
```
The following fields are mapped to the Duo Gateway Application Metadata
- entryPoint: LoginUrl for the SAML Generic Application
- usernameParameter: Default SAML Generic Application value is nameID
- logoutUrl: In Metadata > Logout URL
- decryptionCert: The downloadable certificate in Metadata
- cert: The downloadable certificate in Metadata
- signature_key: Signing key for JWT, must match the same  key value in elasticsearch ROR config
- kibanaExternalHost: The ROR hostname


#### Elasticsearch index in Kibana ROR Dashboard
Make sure to update signature_key in ROR Dashboard with the value. Otherwise you will get JWT errors while login with SAML.


### Login with SAML 2FA enabled
Go to ReadonlyREST Login page (http://ror-deployment.xyz/login) and click Enter using the SAML SSO button. This will redirect to Duo Security Gateway and ask for a two factor code to proceed. Note that the first time it will provision a two factor seed mapped to the user account.

Once Duo authenticates, it will redirect you to the login page with the JWT secure credential created by the SAML response.

### Logout from SAML from ReadonlyREST Enterprise logout button
Click the Logout button from ROR Dashboard. This will redirect you to Duo Gateway logout completion page. Follow the instructions and close the window.
    
