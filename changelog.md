# Changelog

### (2024-08-01) What's new in **ROR 1.59.0**
* **🚀New** (ES) 8.15.0, 7.17.23, 6.7.x support
* **🚀New** (KBN) 7.17.23 support
* **🧐Enhancement** (KBN) Replace a broken Alert and Connectors applications with the link to our (new tool)[https://anaphora.it] for Reports and alerting for Kibana > 8.6.0 (edited) 
* **🐞Fix** (KBN) Handling reporting URL for report generation
* **🐞Fix** (KBN) Embedding with inline JWT is a feature available only in ReadonlyREST PRO and Enterprise
* **🐞Fix** (ES) [Patcher `UnsupportedOperationException` issue on Windows](https://forum.readonlyrest.com/t/ror-1-58-0-for-es8-14-3-windows-setup/2577)
* **🐞Fix** (ES) for the problem with `_async_search` on ES 8.14.x

### (2024-06-30) What's new in **ROR 1.58.0**
* **🚨Security Fix**(KBN) [CVE-2022-39353](https://www.cve.org/CVERecord?id=CVE-2022-39353), [CVE-2020-7753](https://www.cve.org/CVERecord?id=CVE-2020-7753), [CVE-2022-37616](https://www.cve.org/CVERecord?id=CVE-2022-37616), [CVE-2024-29041](https://www.cve.org/CVERecord?id=CVE-2024-29041), [CVE-2022-0691](https://www.cve.org/CVERecord?id=CVE-2022-0691), [CVE-2021-3801](https://www.cve.org/CVERecord?id=CVE-2021-3801), [CVE-2022-25883](https://www.cve.org/CVERecord?id=CVE-2022-25883), [CVE-2022-0512](https://www.cve.org/CVERecord?id=CVE-2022-0512), [CVE-2022-0686](https://www.cve.org/CVERecord?id=CVE-2022-0686), [CVE-2022-0639](https://www.cve.org/CVERecord?id=CVE-2022-0639), [CVE-2022-25881](https://www.cve.org/CVERecord?id=CVE-2022-25881), [CVE-2023-0842](https://www.cve.org/CVERecord?id=CVE-2023-0842), [CVE-2017-16137](https://www.cve.org/CVERecord?id=CVE-2017-16137), [CVE-2022-33987](https://www.cve.org/CVERecord?id=CVE-2022-33987), [CVE-2022-23647](https://www.cve.org/CVERecord?id=CVE-2022-23647), [CVE-2022-36083](https://www.cve.org/CVERecord?id=CVE-2022-36083), [CVE-2024-28176](https://www.cve.org/CVERecord?id=CVE-2024-28176)
* **🚀New** (KBN) [Kibana images with preinstalled ReadonlyREST plugin in Docker Hub](https://hub.docker.com/r/beshultd/kibana-readonlyrest)
* **🚀New** (KBN) 8.14.3, 8.14.2 support
* **🚀New** (ES) 8.14.3, 8.14.2 support
* **🚀New** (ES) ["structured groups" feature](https://github.com/beshu-tech/readonlyrest-docs/blob/develop/details/structured-groups.md) (authorization rules group names and group IDs can be defined separately) 
* **🧐Enhancement** (KBN) New `readonlyrest_kbn.cookies.secure` and `readonlyrest_kbn.cookies.sameSite` cookie settings via kibana.yml
* **🧐Enhancement** (ES) improved error logging on the creation of LDAP connectors
* **🧐Enhancement** (ES) Patcher - invalid state after patching detection improvements
* **🐞Fix** (KBN) Impersonation and session probe logout issue
* **🐞Fix** (KBN) [Problem with the number of replicas and index template, where the number of replicas was always set to 1. Now, the default value will be the same, as in the case of the Kibana index](https://forum.readonlyrest.com/t/0-replicas-for-single-node-clusters/2530)
* **🐞Fix** (KBN) Fix problem with multi-tenancy features when xpack.security.enabled: true

### (2024-05-18) What's new in **ROR 1.57.3**
* **🚨Security Fix** (ES) [CVE-2024-34447](https://nvd.nist.gov/vuln/detail/CVE-2024-34447)
* **🚀New** (KBN) 8.14.1, 8.14.0, 7.17.22 support
* **🚀New** (ES) 8.14.1, 8.14.0, 7.17.22 support
* **🐞Fix** (KBN) The CSRF cookie name issue that caused the “Wrong credentials” error during login
* **🐞Fix** (KBN) Automatic migration issue for Kibana >= 8.8.0 that caused the “mapping set to strict, dynamic introduction of...” error

### (2024-05-05) What's new in **ROR 1.57.2**
* **🚀New** (KBN) 8.13.4, 8.13.3, 7.17.21 support
* **🚀New** (ES) 8.13.4, 8.13.3, 7.17.21 support
* **🐞Fix** (KBN) Kibana <= 7.2.1 doesn’t run
* **🐞Fix** (KBN) Provides a way to migrate an existing session index to the new session
* **🐞Fix** (ES) [Patching issue for Elasticsearch installed from packages](https://forum.readonlyrest.com/t/bootstrap-error-es/2574)
* **🐞Fix** (ES) Patching issue for Elasticsearch OSS versions

### (2024-04-29) What's new in **ROR 1.57.1**
* **🐞Fix** (ES) configuration parsing regression: one group definition can be a string

### (2024-04-28) What's new in **ROR 1.57.0**
* **🚨Security Fix** (ES) [CVE-2024-29025](https://nvd.nist.gov/vuln/detail/CVE-2024-29025)
* **🚀New** (ES) [LDAP Connector](https://docs.readonlyrest.com/elasticsearch#configuration-notes) feature: groups server-side filtering
* **🚀New** (ES) [LDAP Connector](https://docs.readonlyrest.com/elasticsearch#configuration-notes) feature: skip user search option when user attribute is `cn`
* **⚠️Warning** (KBN|ES) Internal API incompatibilities (to take advantage of rolling update capabilities, upgrade ROR KBN first)
* **⚠️Warning** (ES) Support for ES < 6.8.0 was dropped
* **🧐Enhancement** (KBN) User settings available for all access type users
* **🧐Enhancement** (KBN) Add option to change the Default Route and Time zone in User settings
* **🧐Enhancement** (KBN) Provide correlation ID to Kibana logs
* **🧐Enhancement** (ES) Rich, context-based debug logging in the LDAP connector and LDAP-related rules
* **🧐Enhancement** (ES) Additional [validations](https://docs.readonlyrest.com/elasticsearch#configuring-an-acl-with-filter-fields-rules-when-using-kibana): `kibana` rule should not be used with some other rules in the same block
* **🐞Fix** (KBN) Sometimes reports are not generated correctly for Kibana < 8.0.0 and the “Max attempt reached” error appears
* **🐞Fix** (KBN) Adjust interactive API swagger dark mode colors
* **🐞Fix** (KBN) CSRF problem when multiple ECK Kibana instances
* **🐞Fix** (KBN) Plugin doesn’t run for a version Kibana < 7.11.0 when the OIDC proxy is enabled
* **🐞Fix** (KBN) Session probe should log out the user when empty metadata was returned from ES ROR
* **🐞Fix** (ES) Misc issues when `xpack.security.enabled: true` is set
* **🐞Fix** (ES) Patched files permission issue

### (2024-03-15) What's new in **ROR 1.56.0**
* **🚀New** (KBN) Provide a way to switch light/dark mode per user
* **🚀New** (KBN) 8.13.2, 8.13.1, 8.13.0, 7.17.20, 7.17.19 support
* **🚀New** (ES) 8.13.2, 8.13.1, 8.13.0, 7.17.20, 7.17.19 support
* **⚠️Warning** (ES) [for ES > 6.5 patching is required since this version of ROR](https://docs.readonlyrest.com/elasticsearch#id-5.-patch-elasticsearch)
* **🧐Enhancement** (KBN) The activation key will be revalidated in the interval 
* **🧐Enhancement** (KBN) Provide a way to define Activation key [retrieval mode](https://docs.readonlyrest.com/v/develop/universal-builds#change-activation-key-retrieval-mode-via-kibana.yml)
* **🐞Fix** (KBN) Sometimes reports are not generated correctly for Kibana >= 8.0.0 and "Max attempt reached" error  appears 
* **🐞Fix** (KBN) The OIDC scope configuration property was not applied and the default configuration was used instead.
* **🐞Fix** (KBN) The OIDC proxy parameter was not handled properly in case of HTTPs connection over HTTP proxy server
* **🐞Fix** (KBN) Missing information when Kibana is not patched
* **🐞Fix** (ES) [Repositories and Snapshots handling by ES coordinating nodes](https://forum.readonlyrest.com/t/snapshot-status-cannot-modify-incoming-request/2471)
* **🐞Fix** (ES) [Internode SSL `certificate_verification: true` was causing problems with nodes discovery](https://forum.readonlyrest.com/t/upgrade-elasticsearch-8-2-to-8-x-leads-to-ssl-problems/2480)
* **🐞Fix** (ES) Missing `x-elastic-product` header in the response when `fields` and `filter` rules were used
* **🐞Fix** (ES) Proper `forbid` policy handling during processing ROR login request
* **🐞Fix** (ES) `application/nd-json` media type handling (in case of ES `7.x` versions)

### (2024-01-29) What's new in **ROR 1.55.0**
* **🚨Security Fix** (ES) [CVE-2023-51074](https://nvd.nist.gov/vuln/detail/CVE-2023-51074)
* **🚀New** (KBN) 8.12.2 ,8.12.1, 7.17.18, 7.17.17 support
* **🚀New** (ES) 8.12.2, 8.12.1, 7.17.18 support
* **🚀New** (ES) [Elasticsearch images with preinstalled ReadonlyREST plugin in Docker Hub](https://hub.docker.com/r/beshultd/elasticsearch-readonlyrest)
* **🧐Enhancement** (KBN) Optional `readonlyrest_kbn.auth.oidc_kc.proxyURL` kibana.yml configuration for the OIDC connection which allows declaring your proxy URL
* **🧐Enhancement** (KBN) Upon successful activation and edition changes all sessions are cleared and users are logged out
* **🐞Fix** (KBN) Saved objects are not visible for the users on Kibana >= 8.8.0
* **🐞Fix** (ES) [LDAP nested group IDs are properly escaped](https://forum.readonlyrest.com/t/support-kbn-ent-ldap-and-parentheses/2466)
* **🐞Fix** (ES) Logout when a user with restricted `kibana.access` tried to see a restoration status of snapshots in Kibana

### (2023-12-17) What's new in **ROR 1.54.0**
* **🚨Security Fix** (ES) [Scroll API: protected data could leak when the `fields` rule was used with `fls_engine` set to `es` or `es_with_lucene`](https://forum.readonlyrest.com/t/field-rule-not-working-when-exceeding-a-certain-no-of-docs/2415/7)
* **🚀New** (KBN) 8.12.0, 8.11.4 support
* **🚀New** (ES) 8.12.0, 8.11.4, 7.17.17 support
* **🧐Enhancement** (KBN) Provide automatic [cleaning of stale sessions](https://docs.readonlyrest.com/kibana#automatic-session-cleanup)
* **🧐Enhancement** (KBN) Provide automatic cleaning of stale CSRF cookies
* **🐞Fix** (KBN) Adjust the ROR API POST license endpoint body to the contract to respect the `license` body parameter instead of a `token`
* **🐞Fix** (KBN) `CorelationId`` is changed on every session refresh
* **🐞Fix** (ES) ["missing authorization info" problem in some situations when `xpack.security.enabled` was configured to be `true`](https://forum.readonlyrest.com/t/diana-eck/2298/75)

### (2023-11-20) What's new in **ROR 1.53.0**
* **🚨Security Fix** (ES) [CVE-2023-4586](https://nvd.nist.gov/vuln/detail/CVE-2023-4586), [CVE-2023-5072](https://nvd.nist.gov/vuln/detail/CVE-2023-5072)
* **🚀New** (KBN) 8.11.3, 8.11.2, 8.11.1, 8.11.0, 7.17.16 support
* **🚀New** (ES) 8.11.3, 8.11.2, 8.11.1, 8.11.0, 7.17.16 support
* **🧐Enhancement** (KBN) Provide Activate license endpoint to the ReadonlyREST API
* **🧐Enhancement** (ES) [when the `kibana` rule and the `indices` rule are defined in the same block](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#index), there is no need to explicitly allow kibana-related indices
* **🐞Fix** (KBN) problem with reports generation when `kibana.index` in kibana.yml is used
* **🐞Fix** (KBN) crash loop during license service initialization
* **🐞Fix** (KBN) problem with logging in in KBN 7.17.13 (and above) and 8.10.4 (and above) when deployed using ECK
* **🐞Fix** (KBN) [problem with multi-tenancy and ECK](https://forum.readonlyrest.com/t/multi-tanancy-issue/2427)
* **🐞Fix** (KBN) problem with forbidden `/_create/config` response on Login to the Kibana
* **🐞Fix** (ES) [patching fix, when a non-default ES path is used (e.g. on K8s)](https://forum.readonlyrest.com/t/getting-java-lang-illegalargumentexception-when-initializing-ror-in-es-8-10-4/2441)

### (2023-10-09) What's new in **ROR 1.52.0**
* **🚨Security Fix** (ES) [CVE-2023-4586](https://access.redhat.com/security/cve/cve-2023-4586)
* **🚀New** (KBN) 8.10.4, 8.10.3, 7.17.15, 7.17.14 support
* **🚀New** (ES) 8.10.4, 8.10.3, 7.17.15, 7.17.14 support
* **🚀New** (ES) [New `token_authentication` rule](https://docs.readonlyrest.com/elasticsearch#token_authentication)
* **🧐Enhancement** (KBN) Permanently hide Kibana|ES features that are impossible to support
* **🧐Enhancement** (KBN) [License expiration reminder](https://forum.readonlyrest.com/t/license-expiration-reminder/2417)
* **🧐Enhancement** (KBN) Make `kibana.index` setting from kibana.yml an invalid property for an Enterprise user
* **🐞Fix** (KBN) Issue with not adding `elasticsearch.customHeaders` setting from kibana.yml to ROR requests
* **🐞Fix** (KBN) Logout after opening Stack management Upgrading assistant
* **🐞Fix** (KBN) Problem with logging in of two users in two tabs when two Kibana instances are used
* **🐞Fix** (KBN) Problem with logging in when multi-tenancy is enabled and the `indices` rule is defined in the ROR settings

### (2023-09-25) What's new in **ROR 1.51.1**
* **🚨Security Fix** (ES) [`fields` rule didn't work well in the case of ES 7.10.0 and later and more than 10 documents in the response](https://forum.readonlyrest.com/t/field-rule-not-working-when-exceeding-a-certain-no-of-docs/2415)
* **🐞Fix** (KBN) issue with Observability Overview-based applications hiding
* **🐞Fix** (KBN) Correct `kibana.index` handling for KBN >= 7.9.0 when multi-tenancy is disabled or unavailable
* **🐞Fix** (KBN) Unrestricted Kibana Access on the tenancy switch when a selected tenant is not available anymore
* **🐞Fix** (KBN) Unhandled error during login when `multiTenancyEnabled: false`
* **🐞Fix** (ES) LDAP connectivity improvements

### (2023-09-10) What's new in **ROR 1.51.0**
* **🚨Security Fix** (KBN) the issue with [api_only](https://docs.readonlyrest.com/elasticsearch#kibana-related-rules) access level user and accessing via Kibana UI
* **🚀New** (KBN) 8.10.2, 8.10.1, 8.9.2, 7.17.13 support
* **🚀New** (ES) 8.10.2, 8.10.1, 8.10.0, 8.9.2, 7.17.13 support
* **🚀New** (ES) [Dynamic variables transformation support](https://docs.readonlyrest.com/elasticsearch#variables-functions)
* **🧐Enhancement** (KBN) Expose interactive Swagger as a new Security settings tab
* **🧐Enhancement** (KBN) Provide detailed information about the invalid activation key
* **🧐Enhancement** (ES) additional `hide_apps` validation in the `kibana` rule
* **🐞Fix** (KBN) the issue with the persistence of an activation key provided via UI when `readonlyrest_kbn.cookiePass` was not provided. The [readonlyrest_kbn.cookiePass](https://docs.readonlyrest.com/kibana#configuring-kibana) is required `kibana.yml` property
* **🐞Fix** (KBN) issues for Kibana versions between 7.9.0 and 7.10.2, related to the activation key, Spaces, and readonlyREST menu crash
* **🐞Fix** (KBN) The issue with a logout from Kibana when the link to the Kibana is open from a third-party application like `Gmail`
* **🐞Fix** (ES) [getting data streams when not full names of backing indices are declared in the `indices` rule](https://forum.readonlyrest.com/t/forbidden-for-creating-component-templates/2372/7)
* **🐞Fix** (ES) stack-management screen fix in case of `xpack.security.enabled: true`

### (2023-07-25) What's new in **ROR 1.50.0**
* **🚀New** (KBN/ES) ECK support
* **🚀New** (KBN) 8.9.1, 8.9.0, 7.17.12 support
* **🚀New** (ES) 8.9.1, 8.9.0, 7.17.12 support
* **🚀New** (KBN) Introduce the new ReadonlyREST API
* **🧐Enhancement** (KBN) Remove application item info from URL on the tenant switch to avoid a 404 not found message
* **🧐Enhancement** (KBN) Provide Reordering available tenancies for proxy auth authentication
* **🧐Enhancement** (KBN) Provide information about granted/rejected log-in users to debug logs

### (2023-06-27) What's new in **ROR 1.49.1**
* **🚨Security Fix** (ES) [CVE-2023-2976](https://nvd.nist.gov/vuln/detail/CVE-2023-2976)
* **🚨Security Fix** (ES) [CVE-2023-34462](https://github.com/advisories/GHSA-6mjq-h674-j845)
* **🚀New** (KBN) 8.8.2, 8.8.1, 8.8.0, 7.17.11 support
* **🚀New** (ES) 8.8.2, 7.17.11 support
* **🚀New** (ES) [LDAP nested groups support](https://docs.readonlyrest.com/elasticsearch#ldap-connector)
* **🧐Enhancement** (KBN) [Allow setting default tenancy via `/login?defaultGroup` query param. To be used with “Custom Middleware” feature for reordering available tenancies in the ROR menu](https://docs.readonlyrest.com/examples/custom-middleware/reordering-available-tenancies) 
* **🐞Fix** (ES) [Fix for ES warnings in logs about custom action names (ROR internal actions)](https://forum.readonlyrest.com/t/invalid-action-name-cluster-ror-audit-event-put/2186)
* **🐞Fix** (ES) [kibana access `rw` and `admin` should allow to manage component templates](https://forum.readonlyrest.com/t/forbidden-for-creating-component-templates/2372)

### (2023-05-28) What's new in **ROR 1.49.0**
* **🚀New** (ES) 8.8.1 support
* **🧐Enhancement** (KBN) Handle `elasticsearch.serviceAccountSupport` configuration property
* **🧐Enhancement** (KBN) Provide a way to Hidden apps Stack management items hiding
* **🧐Enhancement** (KBN) Provide an automated migration of tenancy indices on major Kibana version upgrade
* **🧐Enhancement** (ES) external group ID patterns support in the external to local groups mapping
* **🐞Fix** (KBN) the issue with the replica number being set to 0 on tenant index creation
* **🐞Fix** (KBN) users won’t log out from Kibana on the 500 status error
* **🐞Fix** (KBN) the issue with Kibana keystore not being read by the Kibana plugin
* **🐞Fix** (KBN < 7.9.0) logging issue when two Kibanas are handled by one browser at the same time
* **🐞Fix** (ES) resolving ENVs to YAML number in ROR settings

### (2023-04-15) What's new in **ROR 1.48.0**
* **🚨Security Fix** (ES) [CVE-2022-45688](https://nvd.nist.gov/vuln/detail/CVE-2022-45688)
* **🚀New** (KBN) 8.7.1, 7.17.10 support
* **🚀New** (ES) 8.8.0, 8.7.1, 7.17.10 support
* **🚀New** (KBN/ES) [Introducing "Custom Middleware" functionality](https://docs.readonlyrest.com/kibana#custom-middleware)
* **🚀New** (KBN/ES) [`allowed_api_paths` support in the `kibana` ACL rule](https://docs.readonlyrest.com/elasticsearch#kibana-related-rules)
* **🚀New** (KBN) Add CSRF protection in the login form
* **🚀New** (KBN) Restore deprecated “kibana.index” support for Kibana > 8.x
* **🚀New** (ES) [all Kibana-related rules are gathered in one, new `kibana` ACL rule](https://docs.readonlyrest.com/elasticsearch#kibana-related-rules)
* **🚀New** (ES) [audit supports a new output type: `log`](https://docs.readonlyrest.com/elasticsearch/audit)
* **🧐Enhancement** (KBN) Provide a way to disable multi-tenancy in ROR Enterprise
* **🧐Enhancement** (KBN) Realign index templates behaviour to the old platform
* **🧐Enhancement** (KBN) Error logs when SAML obtains an unusable username from the assertion
* **🧐Enhancement** (KBN) Test configuration warnings improvement
* **🧐Enhancement** (ES) [Added support to override default response code for not started ROR](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/issues/794)
* **🐞Fix** (KBN) Security card not hidden by default
* **🐞Fix** (KBN) Hidden apps regex with two “or” operators don’t hide all kibana apps
* **🐞Fix** (KBN) Fix Alerting Rules resulting in logout issue
* **🐞Fix** (KBN) Fix audit dashboard
* **🐞Fix** (KBN) Stop handling 500 error from `api/lens/existing_fields`
* **🐞Fix** (KBN) Fix lens app
* **🐞Fix** (KBN < 7.9.x) using a custom kibana index in cooperation with ROR Free

### (2023-02-13) What's new in **ROR 1.47.0**
* **🚨Security Fix** (ES) "/" endpoint was not protected for ES 8.x
* **🚨Security Fix** (ES) "/_cat" endpoint was not protected for all ES versions
* **🚀New** (KBN) 8.7.0, 8.6.2 support
* **🚀New** (ES) 8.7.0, 8.6.2 support
* **🚀New** (ES) [the `data_streams` rule](https://docs.readonlyrest.com/v/develop/elasticsearch#data_streams)
* **🧐Enhancement** (KBN) optimisation in hidden apps feature
* **🐞Fix** (KBN) Opening index management mappings tab forces logout
* **🐞Fix** (KBN) Fix dark mode in the ROR menu
* **🐞Fix** (KBN) YAML editor updates and fixes
* **🐞Fix** (ES) Data streams support in the `indices` rule
* **🐞Fix** (ES) NPE when `_search` with aggregations (script) and the `fields` rule were used together

### (2023-01-02) What's new in **ROR 1.46.0**
* **🚨Security Fix** (ES) [CVE-2022-1471](https://nvd.nist.gov/vuln/detail/CVE-2022-1471), [CVE-2022-41915](https://nvd.nist.gov/vuln/detail/CVE-2022-41915), [CVE-2022-36944](https://nvd.nist.gov/vuln/detail/CVE-2022-36944) in [audit Scala 2.13 jar](https://mvnrepository.com/artifact/tech.beshu.ror/audit)
* **🚀New** (KBN) 8.6.1, 8.6.0, 7.17.9 support
* **🚀New** (ES) 8.6.1, 8.6.0, 7.17.9 support
* **🧐Enhancement** (KBN) Activation key management UI
* **🧐Enhancement** (KBN) Less verbose logging in info mode
* **🧐Enhancement** (KBN) “Stack management” kibana compatibility
* **🐞Fix** (KBN) Test settings pop up won’t show
* **🐞Fix** (KBN) hide apps behaviour when “Management” is hidden
* **🐞Fix** (KBN) Data view with a “:” symbol forces logout from a kibana
* **🐞Fix** (KBN) Session probe causes constant refresh when no `kibana_access` defined
* **🐞Fix** (ES) large report generation using data from a remote cluster with enabled x-pack security

### (2022-12-05) What's new in **ROR 1.45.1**
* **🚀New** (KBN) 8.5.3, 7.17.8 support
* **🚀New** (ES) 8.5.3, 7.17.8 support
* **🐞Fix** (KBN) ROR KBN patching script 

### (2022-11-29) What's new in **ROR 1.45.0**
* **🚨Security Fix** (ES) [CVE-2022-42003](https://nvd.nist.gov/vuln/detail/CVE-2022-42003), [CVE-2022-45146](https://nvd.nist.gov/vuln/detail/CVE-2022-45146)
* **🚀New** (KBN) Activation Key API: read AK from ROR_ACTIVATION_KEY.txt
* **🚀New** (KBN) Activation Key API: submit AK via POST /pkp/license (Basic auth)
* **🚀New** (KBN) Inject CSS/JS files in login page
* **🚀New** (KBN) Add user metadata to <body> for extra UI customization
* **🚀New** (ES) Added groups_and mode to [groups_provider_authorization](https://docs.readonlyrest.com/elasticsearch#groups_provider_authorization) rule
* **🧐Enhancement** (ES) all authorization rules support wildcards in group IDs 
* **🧐Enhancement** (ES) connections in the LDAP pool should not be closed unnecessarily 
* **🧐Enhancement** (KBN) Deterministic reporting index detection
* **🧐Enhancement** (KBN) Move free type impersonation to the local users area
* **🧐Enhancement** (KBN) don’t logout when initial JWT token expires
* **🐞Fix** (KBN) Direct Kibana API requests not aware of kibana_index
* **🐞Fix** (KBN) RO and RO_strict kibana accesses
* **🐞Fix** (ES) [when `fls_engine: es` is configured and `fields` rule is used, aggregations should be available only for allowed fields](https://forum.readonlyrest.com/t/field-level-security-and-aggregations/2133)
* **🐞Fix** (ES) [Data streams creation issue fix](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/issues/829)
* **🐞Fix** (ES) Unknown structure of index settings issue fix
* **🐞Fix** (ES) resolving index names with wildcards should take into consideration the current index state and request indices options

### (2022-10-09) What's new in **ROR 1.44.0**
* **🚨Security Fix** (ES) [CVE-2022-25857](https://nvd.nist.gov/vuln/detail/CVE-2022-25857)
* **🚀New** (KBN) 8.5.2, 8.5.1, 8.5.0, 7.17.7 support
* **🚀New** (ES) 8.5.2, 8.5.1, 8.5.0, 7.17.7 support
* **🚀New** (KBN) **plugin packages are now [universal](https://docs.readonlyrest.com/universal-builds)**
* **🚀New** (KBN) **Manage your activation keys through the [customer portal](https://readonlyrest.com/customer)**
* **🚀New** (ES) Added support for certificates in PEM format
* **🧐Enhancement** (KBN) SAML groups list duplication made header size exceed limits
* **🧐Enhancement** (KBN) kibana_access: admin has now privileges to manage a Kibana cluster
* **🧐Enhancement** (ES) added distributed and persistent Test Settings & Auth Mocks configuration for the Impersonation Feature
* **🧐Enhancement** (ES) handling high load when LDAP rules are used
* **🧐Enhancement** (ES) `client_authentication` settings in internode SSL configuration
* **🧐Enhancement** (ES) `acl:available_groups` dynamic variable can be used in a single value context
* **🐞Fix** (ES) SNI handling (internode SSL)

### (2022-08-22) What's new in **ROR 1.43.0**
* **🚀New** (KBN) 8.4.3, 8.4.2, 8.4.1, 8.4.0, 7.17.6 support
* **🚀New** (ES) 8.4.3, 8.4.2, 8.4.1, 8.4.0, 7.17.6 support
* **🚀New** (KBN) `kibana_custom_js_inject_file` feature
* **🐞Fix** (ES) [`ror-tools` fix for Windows OS (patching ES 3.x issue)](https://forum.readonlyrest.com/t/ror-plugin-for-es-8-x-patch-error/2115)
* **🐞Fix** (ES) resolving indices in the remote x-pack cluster
* **🐞Fix** (KBN|PRO) ROR menu title wraps when version text is too short (cosmetic)
* **🐞Fix** (KBN) infinite loading when kibana_access not defined for user
* **🐞Fix** (KBN) transient error with randomly choosing off range bind port on localhost
* **🐞Fix** (KBN) 404 on login when `xpack.spaces.enabled: false`

### (2022-07-25) What's new in **ROR 1.42.0**
* **🚀New** (KBN|ES) 8.3.3, 8.3.2, 8.3.1, 8.3.0, 7.15.5 support
* **🧐Enhancement** (KBN) Search box in tenancy switcher (when #tenancies > 5)
* **🧐Enhancement** (ES) added configuration warnings in the Impersonation Feature
* **🐞Fix** (KBN) Logout didn’t delete the SAML session on the IdP
* **🐞Fix** (KBN) 5xx errors from Elasticsearch break Kibana users’ session unrecoverably
* **🐞Fix** (ES) ROR node cooperation with X-pack nodes

### (2022-06-21) What's new in **ROR 1.41.0**
* **🚀New** (ES) Added `groups_and` mode to [`ror_kbn_auth`](https://docs.readonlyrest.com/elasticsearch#ror_kbn_auth) and [`jwt_auth`](https://docs.readonlyrest.com/elasticsearch#jwt_auth) rules
* **🧐Enhancement** (KBN) Prevent native credentials dialogue to appear in Kibana when ES responds 401
* **🧐Enhancement** (KBN) Logging in after logout shows the same page you last visited
* **🧐Enhancement** (KBN) x-ror-correlation-id header lets you audit a whole Kibana session
* **🐞Fix** (ES|KBN) tenancy selector didn't work well with `jwt_auth` and `ror_kbn_auth` rules
* **🐞Fix** (KBN) Support for special characters in tenancy names
* **🐞Fix** (KBN) OIDC logout flow redirecting to bad request error
* **🐞Fix** (KBN) OIDC connector not working in Kibana < 7.12.0

### (2022-05-24) What's new in **ROR 1.40.0**
* **🚨Security Fix** (ES) [CVE-2022-25647](https://nvd.nist.gov/vuln/detail/CVE-2022-25647) & [CVE-2022-24823](https://nvd.nist.gov/vuln/detail/CVE-2022-24823) & [CVE-2020-13956](https://nvd.nist.gov/vuln/detail/CVE-2020-13956) & [CVE-2020-36518](https://nvd.nist.gov/vuln/detail/CVE-2020-36518) &  [CVE-2020-13956](https://nvd.nist.gov/vuln/detail/CVE-2020-13956) & [CVE-2020-36518](https://nvd.nist.gov/vuln/detail/CVE-2020-36518)
* **🚨Security Fix** (KBN) “Security” app not entirely hidden in 8.2.x
* **🚀New** (ES) New Support for 8.2.3, 8.2.2, 8.2.1, 7.17.4
* **🚀New** (KBN) New Support for 8.2.2 8.2.1, 7.17.4
* **🚀New** (ES & KBN) [The Impersonation feature](https://docs.readonlyrest.com/kibana#impersonation)
* **🚀New** (ES) [FIPS compliant SSL mode](https://docs.readonlyrest.com/elasticsearch/fips)
* **🧐Enhancement** (KBN) SAML cert is now required
* **🧐Enhancement** (KBN) moved OIDC to better library
* **🧐Enhancement** (KBN) OIDC jwksURL is now required
* **🐞Fix** (ES) `indices: ["1"]` interpreted as integer and fails to parse
* **🐞Fix** (KBN) /login?jwt=xxx authorization now works again
* **🐞Fix** (KBN) OIDC/SAML assertion claims were not forwarded to ES
* **🐞Fix** (KBN) include whitelisted headers while logging
* **🐞Fix** (KBN) basepath handling fixes (too many redirects)
* **🐞Fix** (KBN) Make ROR default space the actual default one
* **🐞Fix** (KBN) OIDC connection error

### (2022-03-19) What's new in **ROR 1.39.0**
* **🚨Security Fix** (KBN) XSS sanitize path requested
* **🚨Security Fix** (ES) [CVE-2020-36518](https://nvd.nist.gov/vuln/detail/CVE-2020-36518) & [CVE-2022-21653](https://nvd.nist.gov/vuln/detail/CVE-2022-21653)
* **🚀New** (KBN) New Support for 8.2.0 8.1.3, 8.1.2, 8.1.1, 8.1.0, 8.0.0, 8.0.1, 7.17.3, 7.17.2
* **🚀New** (ES) New Support for 8.2.0, 8.1.3, 8.1.2, 8.1.1, 8.1.0, 8.0.0, 8.0.1 ([required additional patching step](https://docs.readonlyrest.com/elasticsearch#3.-patch-es))
* **🚀New** (ES) New Support for 7.17.3, 7.17.2
* **🚀New** (ES) [New `groups_and` ACL rule](https://docs.readonlyrest.com/elasticsearch#groups_and)
* **🧐Enhancement** (KBN) Stop inlining whitelisted headers into Authorization header
* **🧐Enhancement** (KBN) Log additional errors and info related to HA
* **🧐Enhancement** (KBN) Misc internal dependencies upgrades 
* **🐞Fix** (KBN) Mandatory elasticsearch credentials in kibana.yml 
* **🐞Fix** (KBN) [Reporting page redirect on refresh when kibana_hide_apps: ["Stack Management"]](https://forum.readonlyrest.com/t/when-hiding-stack-management-a-redirect-appears-with-report/2088)
* **🐞Fix** (KBN) whitelistedPaths: log errors when 404 occurs
* **🐞Fix** (KBN) [Issue uploading large payload](https://forum.readonlyrest.com/t/issue-uploading-large-payload/2091)
* **🐞Fix** (KBN) `elasticsearch.requestHeadersWhitelist` should be case insensitive
* **🐞Fix** (ES) [Issue with handling data streams by `indices` rule](https://forum.readonlyrest.com/t/ror-1-37-0-indices-rule-and-alias-within-kibana/2078)
* **🐞Fix** (ES) X-Pack SSL nodes cooperation with ROR SSL nodes
* **🐞Fix** (ES) _msearch issue when filter rules was used in matched block

### (2022-01-17) What's new in **ROR 1.38.0**
* **🚀New** (ES) New Support for 7.17.0, 7.17.1
* **🚀New** (KBN) New Support for 7.17.0
* **🚀New** (ES) [Configuration for custom audit cluster](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.38.x/elasticsearch.md#custom-audit-cluster)
* **🧐Enhancement** (ES) Separate "audit" section for all audit settings
* **🐞Fix** (KBN) Editor rendering issue with kibana basePath enabled

### (2021-12-14) What's new in **ROR 1.37.0**
* **🚨Security Fix** (ES) [CVE-2021-43797](https://nvd.nist.gov/vuln/detail/CVE-2021-43797)
* **🚀New** (ES) New Support for 7.16.3, 7.16.2, 6.8.23, 6.8.22
* **🚀New** (KBN) New Support for 7.16.3, 7.16.2, 7.16.1, 7.16.10, 6.8.23, 6.8.22, 6.8.21
* **🧐Enhancement** (ES) fields rule handling in the context of x-Pack SQL requests
* **🐞Fix** (ES) filter rule handling in the context of x-Pack SQL requests
* **🐞Fix** (KBN) POST / bulk cause an 400 error in devtools console
* **🐞Fix** (KBN) More robust Kibana patcher + better logs messages

### (2021-11-21) What's new in **ROR 1.36.0**
* **🚀New** (ES) New Support for 7.16.1, 7.16.0, 6.8.21
* **🚀New** (KBN) Support Kibana 7.15.2
* **🚀New** (ES) [Added support for setting up cluster containing ES with ROR (with disabled XPack security) and ES with XPack security enabled](https://forum.readonlyrest.com/t/ssl-internode-with-elk-cluster/1916)
* **🧐Enhancement** (KBN) kibana_hide_apps: [ror|kibana] to remove kibana mgmt button
* **🐞Fix** (ES) [/_snapshot/_status should return only running snapshots](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/issues/756)
* **🐞Fix** (ES) [Adding policy to index template bug](https://forum.readonlyrest.com/t/forbidden-by-readonlyrest-es-plugin-with-add-policy-to-index-template-action-in-kibana/1969)
* **🐞Fix** (KBN) Index management tabs result in "forbidden" error
* **🐞Fix** (KBN) [corrupted patch file for Kibana 7.9.x](https://forum.readonlyrest.com/t/ror-1-35-1-kibana-7-9-3-unable-to-patch/2018)
* **🐞Fix** (KBN) [YAML editor not working in air-gapped environments](https://forum.readonlyrest.com/t/readonlyrest-security-settings-editor-loading/2014/5)
* **🐞Fix** (KBN) [Devtools not working](https://forum.readonlyrest.com/t/kibana-devtools-error-does-not-support-having-a-body/2027)
* **🐞Fix** (KBN) [Monitoring not working in multi-tenancy](https://forum.readonlyrest.com/t/kibana-alerting-not-working-with-readonlyrest/1986)
* **🐞Fix** (KBN) Regression in Kibana < 6.8.x front end crash
* **🐞Fix** (KBN) Kibana < 7.8.x prevent navigation to hidden apps from home links
* **🐞Fix** (KBN) Kibana < 7.8.x implicitly hide kibana:dashboard when kibana:dashboards is hidden (and viceversa)
* **🐞Fix** (KBN) Kibana < 7.8.x broken `clearSessionOnEvents: [tenancyHop]`

### (2021-10-17) What's new in **ROR 1.35.1**
* **🚨Security Fix** (ES) [CVE-2021-21409](https://nvd.nist.gov/vuln/detail/CVE-2021-21409) & [CVE-2021-27568](https://nvd.nist.gov/vuln/detail/CVE-2021-27568)
* **🚀New** (KBN) Support Kibana 7.15.1
* **🚀New** (ES) New Support for 7.15.2
* **🧐Enhancement** (KBN) Support "server.ssl.supportedProtocols" settings
* **🧐Enhancement** (KBN) Support "server.ssl.cipherSuites"
* **🧐Enhancement** (KBN) Always honor SSL cipher order
* **🐞Fix** (KBN) Don'thide "Add/Remove field as column" in Discover app for RO users
* **🐞Fix** (KBN) More alerting fixes (only for main tenancy)

### (2021-10-12) What's new in **ROR 1.35.0**

* **🚀New** (KBN) Support Kibana 7.15.0, 7.14.2
* **🚀New** (ES) New Support for 7.15.1, 6.8.19, 6.8.20
* **🧐Enhancement** (ES) [local->external groups detailed mapping for groups rule](https://github.com/beshu-tech/readonlyrest-docs/blob/master/details/groups-rule-mapping.md)
* **🧐Enhancement** (ES) when ROR is starting any request is going to end up with HTTP 403 response, instead of HTTP 503
* **🧐Enhancement** (KBN) "server.basePath" kibana option implementation
* **🧐Enhancement** (KBN) Support full regex in kibana_hidden_apps rule
* **🧐Enhancement** Crash if Kibana is not patched
* **🧐Enhancement** (KBN) Honour kibana setting "logging.dest"
* **🧐Enhancement** (KBN) Confirm before overwriting audit log dashboard
* **🐞Fix** (ES) verbosity: error fix in case of ROR KBN login request
* **🐞Fix** (KBN) Make alerting work on primary tenancy
* **🐞Fix** (KBN) OIDC fix sameSite / secure cookie options
* **🐞Fix** (KBN) Login form is stretched when long error
* **🐞Fix** (KBN) Login form is stretched when long error
* **🐞Fix** (KBN-PRO) [Don't send x-ror-currentgroup in PRO](https://forum.readonlyrest.com/t/upgrading-6-7-w-1-18-to-7-14-w-1-33-ldap-from-ms-active-directory-no-longer-understands-multiple-ad-group-memberships/1973/6)
* **🐞Fix** (KBN) Resolve browser console errors on a popover close

### (2021-09-24) What's new in **ROR 1.34.0**

* **🚀New** (ES) New Support for 7.15.0, 7.14.2
* **🚀New** (KBN) VS Code style YAML editor
* **🚀New** (KBN) Skip rendering hidden app groups entirely
* **🚀New** (KBN) Redesigned ROR Menu
* **🚀New** (KBN) Dark theme awareness
* **🐞Fix** (KBN) Broken Kibana Spaces
* **🐞Fix** (KBN) Support Kibana's undocumented "server.ssl.*" settings
* **🐞Fix** (KBN) cookiePass config parsing broke load balancing

### (2021-08-14) What's new in **ROR 1.33.1**

* **🚀New** (ES) New Support for 7.14.1
* **🐞Fix** (KBN) Error in patching for 7.14.0
* **🐞Fix** (KBN) clearSessionOnEvents now works as expected
* **🐞Fix** (KBN) login form font loads correctly

### (2021-08-09) What's new in **ROR 1.33.0**

* **🚨Security Fix** (KBN) xml-crypto dependency update
* **🚀New** (KBN) New Support for 7.14.0, 6.8.18
* **🧐Enhancement** (KBN) Parse credentials in /api/* requests, no need for valid cookie. Supersedes whitelistedPaths
* **🐞Fix** (KBN)Caching issues switching tenancies with dark/light theme
* **🐞Fix** (KBN) Newly created Space shows in all tenancies when using default kibana index
* **🐞Fix** \(KBN &lt; 7.9.x\) nextUrl works again with SAML and OIDC

### (2021-07-25) What's new in **ROR 1.32.0**

* **🚨Security Fix** (ES) [Apache Commons Codec vulnerability](https://forum.readonlyrest.com/t/security-vulnerability-for-common-codec-1-10/1906)
* **🚨Security Fix** (KBN) upgraded dependencies due to security fixes
* **🚨Security Fix** (KBN) disable x-powered-by to avoid fingerprinting
* **🚀New** (ES) Support for ES 7.14.0 & 6.8.18
* **🚀New** (KBN) Support for Kibana 7.13.x series
* **🧐Enhancement** (KBN) honor configurations coming from ENV and CLI options
* **🧐Enhancement** (KBN) when metadata has no username, login must be denied
* **🧐Enhancement** (KBN) audit tab ported to new platform
* **🧐Enhancement** (ES) improved ES resources cleaning when ROR returns FORBIDDEN response
* **🧐Enhancement** \(KBN &lt; 7.9.x\) auto clean-up dangling SAML/OIDC cookies
* **🐞Fix** (ES) [incomplete response for request GET */_alias](https://forum.readonlyrest.com/t/ror-return-incomplete-response-for-request-get-alias/1872)
* **🐞Fix** (ES) not allowed aliases should not present in a response for a Get Index API request
* **🐞Fix** (KBN) fix dev-tools and import saved object not working
* **🐞Fix** (KBN) honor `requestHeadersWhitelist` in user metadata request (login)
* **🐞Fix** \(KBN &lt; 7.9.x\) do not crash on invalid metadata
 
### (2021-06-29) What's new in **ROR 1.31.0**

* **🚨Security Fix** (KBN) prevent direct navigation to hidden apps
* **🚀New** (ES) 7.13.4, 7.13.3, 7.13.2, 6.8.17 support
* **🚀New** (KBN) new minimal Kibana Management menu when "Management" app is hidden
* **🧐Enhancement** (KBN) logout active Kibana session if key metadata/permissions change in ACL
* **🧐Enhancement** (KBN) better port number validation
* **🧐Enhancement** (ES) improved cluster indices handling
* **🐞Fix** (ES) [Kibana access rule regression fix](https://forum.readonlyrest.com/t/es7-11-2-1-30-0-enterprise-two-contexts-rw-ro-issue/1855)
* **🐞Fix** (ES) search template API handling with `filter` and `fields` rule
* **🐞Fix** (ES) multi-tenancy issue when groups_provider_authorization is used
* **🐞Fix** (ES) `x_forwarded_for` rule: wrong handling of / request
* **🐞Fix** (ES) Issue with handling ResizeRequest which made it unable to upgrade Kibana to version 7.12.0+
* **🐞Fix** (KBN) some Kibana requests arrive to ES without credentials
* **🐞Fix** (KBN) inconsistent read after write in session storage lead to issues with round robin load balancing
* **🐞Fix** (KBN) bad multipart POST handling leads to saved object import errors

### (2021-05-26) What's new in **ROR 1.30.1**

* **🚨Security Fix** \(ES\) [CVE-2021-27568](https://nvd.nist.gov/vuln/detail/CVE-2021-27568)
* **🚀New** \(ES\) 7.13.0, 7.13.1 support
* **🐞Fix** \(ES\) Regression in multi-tenancy handling
* **🐞Fix** \(ES\) Proper handling of \_snapshot/\_status endpoint

### (2021-05-16) What's new in **ROR 1.30.0**

* **🚀New** \(KBN\) 7.12.x compatibility
* **🚀New** \(ES\) [LDAP connector circuit breaker](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.30.x/elasticsearch.md#circuit-breaker)
* **🧐Enhancement** \(ES\) [Username with wildcard support in users section](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.30.x/elasticsearch.md#groups) and [groups mapping](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.30.x/elasticsearch.md#group-mapping)
* **🧐Enhancement** \(KBN &lt; 7.9.x\) OIDC errors visibility
* **🧐Enhancement** \(KBN &lt; 7.9.x\) Smarter session probe algorithm
* **🐞Fix** \(KBN &gt;= 7.9.x\) [Load CertificateAuthorities as an array if not specified as an array](https://forum.readonlyrest.com/t/kibana-crash-at-startup-with-the-new-7-10-2-version/1840)
* **🐞Fix** \(KBN &lt; 7.9.x\) Don't hide visualizations list search box in RO mode

### (2021-04-09) What's new in **ROR 1.29.0**

* **🚨Security Fix** \(ES\) Security Fix \(ES\) [CVE-2021-21409](https://nvd.nist.gov/vuln/detail/CVE-2021-21409)
* **🚀New** \(KBN\) support 7.9.0, 7.9.1, 7.10.0, 7.10.1, 7.10.2, 7.11.0, 7.11.1, 7.11.2 \([with ROR new platform](https://beta.readonlyrest.com/)\) 
* **🚀New** \(ES\) 7.12.1 support 
* **🧐Enhancement** \(KBN\) logout if the credentials/metadata of the current user change in the ACL

### (2021-04-01) What's new in **ROR 1.28.2**

* **🚨Security Fix** \(ES\) [CVE-2021-21295](https://nvd.nist.gov/vuln/detail/CVE-2021-21295)
* **🐞Fix** \(KBN\) prevent SAML/OIDC initiated Kibana sessions from expiring after `session_timeout_minutes` despite continued interaction

### (2021-03-24) What's new in **ROR 1.28.1**

* **🐞Fix** \(ES\) Getting index templates issue when no `indices` rule was used in matched block
* **🐞Fix** \(ES\) [NPE on getting template aliases](https://forum.readonlyrest.com/t/cannot-put-index-template-template-1/1681/25)

### (2021-03-14) What's new in **ROR 1.28.0**

* **🚀New** \(ES\) 7.12.0, 7.11.2 support 
* **🚀New** \(ES\) full [Index and Component Templates API](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/index-templates.html) support 
* **🧐Enhancement** \(ES\) [Username case sensitivity settings](https://forum.readonlyrest.com/t/ldap-based-user-authentication/1667)
* **🐞Fix** \(ES\) [Kibana logout event storing fix](https://forum.readonlyrest.com/t/kibana-plugin-software-licensing-and-expiration/1808/5)
* **🐞Fix** \(ES\) [Fixed remote reindex operation with "type" parameter](https://forum.readonlyrest.com/t/reindex-index-not-found-exception/1708/20)
* **🐞Fix** \(KBN\) Prevent cookie expiration deadlock in browsers when using SAML/OIDC
* **🐞Fix** \(KBN\) When credentials change in the ACL, make it possible to login again
* **🐞Fix** \(KBN\) Kibana management app ID changed from "kibana:management" to "kibana:stack\_management"

### (2021-02-27) What's new in **ROR 1.27.1**

* **🚨Security Fix** \(ES\) [CVE-2021-21290](https://nvd.nist.gov/vuln/detail/CVE-2021-21290)
* **🚀New** \(ES\) 7.11.1 support 

### (2021-02-16) What's new in **ROR 1.27.0**

* **🚀New** \(ES\) 7.11.0, 7.10.2, 6.8.14 support
* **🧐Enhancement** \(KBN\) X-Forwarded-For copied from incoming request \(or filled with source IP\) before forwarding to ES
* **🧐Enhancement** \(KBN\) Kibana logout event generates a special audit log entry in ROR audit logs index
* **🧐Enhancement** \(KBN\) ROR panel shows "reports" button if kibana:management app is hidden
* **🐞Fix** \(ES\) [blocks containing filter and/or fields won't match internal kibana requests, so kibana\_\* rules won't have to be placed in such blocks](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#fields)
* **🐞Fix** \(ES\) SQL API - better handling of invalid query

### (2021-01-11) What's new in **ROR 1.26.1**

* **🐞Fix** \(ES\) wrong behaviour of `kibana_access` rule for ROR actions when ADMIN value is set

### (2021-01-02) What's new in **ROR 1.26.0**

* **🚨Security Fix** \(ES\) [CVE-2020-35490](https://nvd.nist.gov/vuln/detail/CVE-2020-35490) & [CVE-2020-35490](https://nvd.nist.gov/vuln/detail/CVE-2020-35491) \(removed Jackson dependency from ROR core\)
* **🚀New** \(ES\) [New response\_fields rule](https://forum.readonlyrest.com/t/ror-1-18-9-enterprise-es-7-2-0-enable-cluster-health-without-authentication/1567)
* **🚀New** \(ES\) [Support for LDAP server discovery using \_ldaps.\_tcp SRV record](https://forum.readonlyrest.com/t/does-ror-support-dc-locator/1211)
* **🚀 New** \(ES\) [New configuration option allowing to ignore LDAP connectivity problems](https://forum.readonlyrest.com/t/ror-cannot-start-if-ldap-is-not-available/1748)
* **🧐Enhancement** \(ES\) Full support for ILM API
* **🧐Enhancement** \(KBN\) Enforce read-after-write consistency between kibana nodes
* **🧐Enhancement** \(KBN ENT\) OIDC custom claims incorporated in "assertion" claim
* **🧐Enhancement** \(KBN ENT\) OIDC support for configurable kibanaExternalHost \(good for Docker\)
* **🧐Enhancement** \(KBN ENT\) ROR adds "ror-user\_" class to "body" tag for easy per-user CSS/JS
* **🧐Enhancement** \(KBN ENT/PRO\) ROR adds "ror-group\_" class to "body" tag for easy per-group CSS/JS
* **🐞Fix** \(ES\) [ROR authentication endpoint action](https://forum.readonlyrest.com/t/es-7-4-2-ror-1-18-9-rradmin-refreshsettings-by-block-default/1388)
* **🐞Fix** \(ES\) "username" in audit entry when request is rejected

### What's new in 1.25.2

* **🐞Fix** \(ES\) [removed verbose logging](https://forum.readonlyrest.com/t/elastic-message-cannot-extract-fields-for-query-after-readonlyrest-installation/1749)

### What's new in 1.25.1

* **🚨Security Fix** \(ES\) [CVE-2020-25649](https://nvd.nist.gov/vuln/detail/CVE-2020-25649)
* **🚀New** \(ES\) 7.10.1 support

### What's new in 1.25.0

* **🚨Security Fix** \(ES\) [Common Vulnerabilities and Exposures \(CVE\)](https://forum.readonlyrest.com/t/update-of-jackson-databind-2-9-6-jar/176)
* **🚀New** \(ES\) 7.10.0 support
* **🚀New** \(ES\) [auth\_key\_pbkdf2 rule](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.25.x/elasticsearch.md#auth_key_pbkdf2)
* **🚀New** \(ES\) [Introduced configuration property defining FLS engine used by fields rule](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.25.x/elasticsearch.md#fields)
* **🧐Enhancement** \(ES\) Fields rule performance improvement
* **🧐Enhancement** \(ES\) Resolved index API support
* **🐞Fix** \(ES\) ["username" in audit entry when user is authenticated via proxy\_auth](https://forum.readonlyrest.com/t/ror-audit-not-logging-user-id)
* **🐞Fix** \(ES\) index resolve action should be treated as readonly action
* **🐞Fix** \(ES\) /\_snapshot and /\_snapshot/\_all should behave the same

### What's new in 1.24.0

* **🚨Security Fix** \(ES\) search template handling fix
* **🚀New** \(ES\) 7.9.3 & 6.8.13 support
* **🧐Enhancement** \(ES\) full support for ES Snapshots and Restore APIs
* **🐞Fix** \(KBN\) fix crash in error handling
* **🐞Fix** \(ES\) don't remove ES response warning headers
* **🐞Fix** \(ES\) issue when entropy of /dev/random could have been exhausted when using JwtToken rule

### What's new in 1.23.1

* **🚀New** \(ES\) 7.9.2 support
* **🐞Fix** \(KBN\) fix code 500 error on login in Kibana

### What's new in 1.23.0

* **🚀New** \(ES\) introduced must\_involve\_indices option for indices rule
* **🧐Enhancement** \(ES\) negation support in headers rules
* **🧐Enhancement** \(ES\) [x-pack rollup API handling](https://forum.readonlyrest.com/t/actions-still-forbidden-to-unrestricted-user/1659)
* **🐞Fix** \(KBN\) deep links query parameters are now handled
* **🐞Fix** \(KBN\) make sure default kibana index is always discovered \(fixes reporting in 6.x\)
* **🐞Fix** \(ES\) [settings file permission issue with JDK 1.8.0 25.262-b10](https://forum.readonlyrest.com/t/readonlyrest-for-elastic-wont-start-1-18-8-es6-8-1/1652)
* **🐞Fix** \(ES\) /\_cluster/allocation/explain request should not be forbidden if matched block doesn't have indices rules
* **🐞Fix** \(ES\) remote address extracting issue
* **🐞Fix** \(ES\) [fixed TYP audit field for some request types](https://forum.readonlyrest.com/t/match-wrong-index-in-forbid-block/1653/2)

### What's new in 1.22.1

* **🐞Fix** \(ES\) missing handling of aliases API for ES 7.9.0

### What's new in 1.22.0

* **🚀New** \(ES\) 7.9.0 support
* **🧐Enhancement** \(ES\) aliases API handling
* **🧐Enhancement** \(ES\) dynamic variables support in fields rule
* **🐞Fix** \(ES\) [adding aliases issue](https://forum.readonlyrest.com/t/actions-still-forbidden-to-unrestricted-user/1659)
* **🐞Fix** \(ES\) potential memory leak for ES 7.7.x and above
* **🐞Fix** \(ES\) cross cluster search issue fix for X-Pack \_async\_search action
* **🐞Fix** \(ES\) XFF entry in audit issue
* **🐞Fix** \(KBN\) SAML certificate loading
* **🐞Fix** \(KBN\) SAML loading groups from assertion
* **🐞Fix** \(KBN\) fix reporting in pre-7.7.0

### What's new in 1.21.0

* **🧐Enhancement** \(ES\) [cluster API support improvements](https://forum.readonlyrest.com/t/settings-problems/1616)
* **🐞Fix** \(ES\) X-Pack \_async\_search support
* **🐞Fix** \(ES\) \_rollover request handling
* **🐞Fix** \(ES\) [handling numeric ssl configuration properties](https://forum.readonlyrest.com/t/numeric-passphrases-invalid-ssl-config/1512)
* **🐞Fix** \(KBN\) multitenancy+reporting regression fix \(for 7.6.x and earlier\)
* **🐞Fix** \(KBN\) "x-" headers should be forwarded in /login route when proxy passthrough is enabled
* **🐞Fix** [\(KBN\) Logout now redirects to login screen when using proxy](https://forum.readonlyrest.com/t/kibana-ror-1-19-5-issue/1576/24)
* **🐞Fix** \(KBN\) SAML metadata.xml endpoint not responding
* **🐞Fix** \(KBN\) NAT/reverse proxy support for SAML
* **🐞Fix** \(KBN\) SAML login redirect error
* **🐞Fix** \(ES\) \_readonlyrest/metadata/current\_user should be always allowed by filter/fields rule

### What's new in 1.20.0

* **🚀New** 7.7.1, 7.8.0 support
* **🧐Enhancement** \(KBN\) tidy up audit page
* **🧐Enhancement** \(KBN FREE\) clearly inform when features are not available
* **🧐Enhancement** \(KBN\) ship license report of libraries
* **🧐Enhancement** \(ES\) filter rule performance improvement
* **🐞Fix** \(KBN\) proxy\_auth: avoid logout-login loop
* **🐞Fix** \(KBN\) 404 error on font CSS file
* **🐞Fix** \(ES\) [wildcard in filter query issue](https://forum.readonlyrest.com/t/wildcard-in-dls-filter-gives-error/1551)
* **🐞Fix** \(ES\) [forbidden /\_snapshot issue](https://forum.readonlyrest.com/t/get-snapshot-permission-issue/1594)
* **🐞Fix** \(ES\) /\_mget handling by indices rule when no index from a list is found
* **🐞Fix** \(ES\) available groups order in metadata response should match the order in which groups appear in ACL
* **🐞Fix** \(ES\) .readonlyrest and audit index - removed usage of explicit index type
* **🐞Fix** \(ES\) [tasks leak bug](https://forum.readonlyrest.com/t/lots-of-active-tasks-in-cat-tasks/1593)

### What's new in 1.19.5

* **🚀New** 7.7.0, 7.6.2, 6.8.9, 6.8.8 support
* **🧐Enhancement** \(ES/KBN\) kibana\_access can be explicitly set to unrestricted
* **🧐Enhancement** \(ES\) [LDAP connection pool improvement](https://forum.readonlyrest.com/t/losing-connections-to-ldap-servers/1485)
* **🐞Fix** \(ES\) [better LDAP request timeout handling](https://forum.readonlyrest.com/t/losing-connections-to-ldap-servers/1485)
* **🐞Fix** \(ES\) remote indices searching bug
* **🐞Fix** \(ES\) cross cluster search support for \_field\_caps request
* **🚨Security Fix** \(ES\) create and delete templates handling
* **🐞Fix** \(KBN\) Regression in proxy\_auth\_passthrough
* **🧐Enhancement** \(KBN\) whitelistedPaths now accepts basic auth credentials
* **🧐Enhancement** \(KBN\) Dump logout button, [new ROR Panel](https://forum.readonlyrest.com/t/new-logout-button-design-new-ror-panel/1476)
* **🧐Enhancement** \(KBN\) removed ROR from Kibana sidebar. Admins have a link in new panel.
* **🧐Enhancement** \(KBN\) avoid show login form redirecting from SAML IdP
* **🚀New** \(KBN\) [OpenID Connect \(OIDC\) authentication connector](https://github.com/beshu-tech/readonlyrest-docs/blob/master/kibana.md#openid-connect-oidc)
* **🚀New** \(KBN\) [login\_title, login\_subtitle enable 2 column login page](https://forum.readonlyrest.com/t/ror-enterprise-show-support-contact-on-login-page/1508/2)
* **🚨Security Fix** \(KBN\) server-side navigation prevention to hidden apps

### What's new in 1.19.4

* **🐞Fix** \(ES\) Interpolating config with environment variables in SSL section
* **🐞Fix** \(KBN Ent 6.x\) Fixed default space creation in
* **🐞Fix** \(KBN 6.x\) Fixed error toast notification not showing
* **🐞Fix** \(KBN Ent\) Fixed missing Axios dependency
* **🐞Fix** \(KBN Ent\) Fixed SAML connector
* **🐞Fix** \(KBN\) Toast notification overlap with logout bar
* **🧐Enhancement** \(KBN\) Restyled logout bar
* **🧐Enhancement** \(KBN\) Configurable periodic session checker

### What's new in 1.19.3

* **🚀New** \(ES/KBN\) 7.6.1 compatibility
* **🚀New** \(ES\) customizable name of settings index
* **🧐Enhancement** \(KBN\) configurable ROR cookie name
* **🧐Enhancement** \(ES/KBN\) handling of encoded ROR headers in Authorization header values
* **🧐Enhancement** \(KBN\) user feedback on why login failed
* **🐞Fix** \(ES\) support for multiple header values
* **🐞Fix** \(ES\) releasing LDAP connection pool on reloading ROR settings
* **🐞Fix** \(KBN\) multitenancy issue with 7.6.0+
* **🐞Fix** \(KBN\) creation of default space for new tenant
* **🐞Fix** \(KBN 6.x\) in RO mode, don't hide add/remove over fields in discovery
* **🐞Fix** \(KBN 6.x\) index template & in-index session manager issues

### What's new in 1.19.2

* **🚀New** \(KBN\) 7.6.0 support
* **🧐Enhancement** \(KBN\) less verbose info logging
* **🧐Enhancement** \(KBN\) start up time semantic check for settings
* **🐞Fix** \(KBN Free\) missing logout button
* **🐞Fix** \(KBN\) error message creating internal proxy
* **🐞Fix** \(KBN 6.x\) add field to filter button invisible in RO mode

### What's new in 1.19.1

* **🎁Product** \(KBN\) [Launched ReadonlyREST Free for Kibana!](https://forum.readonlyrest.com/t/provide-kibana-login-page-for-ror-oss-version/1441/2?u=sscarduzio)
* **🚀New** \(ES\) 7.6.0 support, Kibana support coming soon
* **🚀New** \(KBN\) Audit log dashboard
* **🚀New** \(KBN\) Template index can now be declared per tenant instead of globally
* **🚀New** \(ES\) custom trust store file and password options in ROR settings
* **🧐Enhancement** \(ES\) When "prompt\_for\_basic\_auth" is enabled, ROR is going to return 401 instead of 404 when the index is not found or a user is not allowed to see the index
* **🧐Enhancement** \(ES\) literal ipv6 with zone Id is acceptable network address
* **🧐Enhancement** \(ES\) LDAP client cache improvements
* **🐞Fix** \(ES\) /\_all/\_settings API issue
* **🐞Fix** \(ES\) Index stats API & Index shard stores API issue
* **🐞Fix** \(ES\) readonlyrest.force\_load\_from\_file setting decoding issue
* **🐞Fix** \(KBN\) allowing user to be logged in in two tabs at the same time
* **🐞Fix** \(KBN\) logging with JWT parameter issue
* **🐞Fix** \(KBN\) parsing of sessions fetched from ES index
* **🐞Fix** \(KBN\) logout issue

### What's new in 1.19.0

* **🚀New** \(KBN\) Configurable option to delete docs from tenant index when not present in template
* **🧐Enhancement** \(ES\) Less verbose logging of blocks history
* **🧐Enhancement** \(ES\) Enriched logs and audit with attempted username
* **🧐Enhancement** \(ES\) Better settings validation - only one authentication rule can be used in given block
* **🧐Enhancement** \(ES/KBN\) Plugin versions printing in logs on launch
* **🧐Enhancement** \(ES\) When user doesn't have access to given index, ROR pretends that the index doesn't exist and return 404 instead of 403
* **🐞Fix** \(ES\) Searching for nonexistent/forbidden index with wildcard mirrors default ES behaviour instead of returning 403
* **🐞Fix** \(KBN\) Switching groups bug

### What's new in 1.18.10

* **🚀New** \(ES/KBN\) Support v6.8.6, v7.5.0, v7.5.1
* **🚀New** \(KBN\) Group IDs can now be mapped to aliases
* **🚀New** \(ES\) New, more robust and simple method of creating custom audit log serializers
* **🚀New** \(ES\) Example projects with custom audit log serializers
* 🐞**Fix** \(KBN\) Prevent index migration after kibana startup
* **🧐Enhancement** \(KBN\) If default space doesn't exist in kibana index then copy from default one
* **🧐Enhancement** \(KBN\) Crypto improvements - store init vector with encrypted data as base64 encoded json.
* **🧐Enhancement** \(ES\) Better settings validation - prevent duplicated keys in readonlyrest.yml

### What's new in 1.18.9

* **🚀New** \(ES/KBN\) Support v7.4.1, v7.4.2
* **🚀New** \(KBN\) Kibana sessions stored in ES index
* 🐞**Fix** \(ES\) issue with in-index settings auto-reloading
* 🐞**Fix** \(ES\) \_cat/indices empty response when matched block doesn't contain 'indices' rule

### What's new in 1.18.8

* **🚀New** \(ES/KBN\) Support v7.4.0
* **🚀New** \(ES\) Elasticsearch SQL Support
* **🚀New** \(ES\) Internode ssl support for es5x, es60x, es61x and es62x
* **🚀New** \(ES\) new runtime variable @{acl:current\_group}
* **🚀New** \(ES\) namespace for user variable and support for both versions: @{user} and @{acl:user}
* **🚀New** \(ES\) support for multiple values in uri\_re rule
* **🧐Enhancement** \(ES\) more reliable in-index settings loading of ES with ROR startup
* **🧐Enhancement** \(ES\) less verbose logs in JWT rules
* **🧐Enhancement** \(ES\) Better response from ROR API when plugin is disabled
* **🧐Enhancement** \(ES\) Splitting verification ssl property to client\_authentication and certificate\_verification
* **🐞Fix** \(ES\) issue with backward compatibility of proxy\_auth settings
* **🐞Fix** \(ES\) /\_render/template request NPE
* **🐞Fix** \(ES\) \_cat/indices API bug fixes
* **🐞Fix** \(ES\) \_cat/templates API return empty list instead of FORBIDDEN when no indices are found
* **🐞Fix** \(ES\) updated regex for kibana access rule to support 7.3 ES
* **🐞Fix** \(ES\) proper resolving of non-string ENV variables in readonlyrest.yml
* **🐞Fix** \(ES\) lang-mustache search template handling

### What's new in 1.18.7

* **🚀New** \(ES\) Field level security \(FLS\) supports nested JSON fields
* **🐞Security Fix** \(ES\) Authorization headers appeared in clear in logs
* **🧐Enhancement** \(KBN\) Don't logout users when they are not allowed to search a index-pattern
* **🧐Enhancement** \(ES\) Headers obfuscation is now case insensitive

### What's new in 1.18.6

* **🚀New** \(ES/KBN\) Support v7.3.1, v7.3.2
* **🚀New** \(ES\) Configurable header names whose value should be obfuscated in logs
* **🚀New** \(KBN\) Dynamic variables from user identity available in custom\_logout\_link
* **🧐Enhancement** \(ES\) Richer logs for JWT errors
* **🧐Enhancement** \(ENT\) nextUrl works also with SAML now
* **🧐Enhancement** \(ENT\) SAML assertion object available in ACL dynamic variables
* **🧐Enhancement** \(KBN\) Validate LDAP server\(s\) before accepting new YAML settings
* **🧐Enhancement** \(KBN\) Ensure a read-only UX for 'ro' users in older Kibana
* **🐞Fix** \(ES\) Fix memory leak from dependency \(snakeYAML\)

### What's new in 1.18.5

* **🐞Security Fix** \(ES\) indices rule can now properly handle also the templates API
* **🧐Enhancement** \(ES\) Array dynamic variables are serialized as CSV wrapped in double quotes
* **🧐Enhancement** \(ES\) Cleaner debug logs \(no stacktraces on forbidden requests\)
* **🧐Enhancement** \(ES\) LDAP debug logs fire also when cache is hit
* **🚀New** \(ES/KBN\) Support v7.2.1, v7.3.0
* **🐞Fix** \(PRO\) PRO plugin crashing for some Kibana versions
* **🐞Fix** \(ENT\) SAML library wrote a too large cookie sometimes
* **🐞Fix** \(ENT\) SAML logout not working
* **🐞Fix** \(ENT\) JWT fix exception "cannot set requestHeadersWhitelist"
* **🐞Fix** \(PRO/ENT\) Hide more UI elements for RO users
* **🐞Fix** \(PRO/ENT\) Sometimes not all the available groups appear in tenancy selector
* **🐞Fix** \(PRO/ENT\) Feature "nextUrl" broke
* **🐞Fix** \(PRO/ENT\) prevent user kick-out when APM is not configured and you are not an admin
* **🚀New** \(PRO/ENT\) Kibana request path/method now sent to ES \(good for policing dev-tools\)

### What's new in 1.18.4

* **🚀New** \(ES\) User impersonation API
* **🚀New** \(ES\) Support latest 6.x and 5.x versions
* **🐞Security Fix** \(ES\) filter/fields rules leak
* **🐞Fix** \(KBN/ENT\) allow more action for kibana\_access, prevent sudden logout
* **🐞Fix** \(KBN/ENT\) temporarily roll back "support for unlimited tenancies"

### What's new in 1.18.3

* **🚀New** Support added for ES/Kibana 6.8.1
* **🧐Enhancement** \(ES\) Crash ES on invalid settings instead of stalling forever
* **🧐Enhancement** \(ES\) Better logging on JWT, JSON-paths, LDAP, YAML errors
* **🧐Enhancement** \(ES\) Block level settings validation to user with precious hints
* **🧐Enhancement** \(ES\) If force\_load\_from\_file: true, don't poll index settings
* **🧐Enhancement** \(ES\) Order now counts declaring LDAP Failover HA servers
* **🐞Fix** \(ES\) "EsIndexJsonContentProvider" had a null pointer exception
* **🐞Fix** \(ES\) "es.set.netty.runtime.available.processors" exception
* **🧐Enhancement** \(KBN\) Collapsible logout button
* **🧐Enhancement** \(KBN\) ROR App now uses a HA http client
* **🧐Enhancement** \(KBN\) Automatic logout for inactivity
* **🧐Enhancement** \(KBN\) Support unlimited amount of tenancies
* **🐞Fix** \(KBN/ENT\) concurrent multitenancy bug
* **🐞Fix** \(KBN\) Avoid sporadic errors on Save/Load buttons

### What's new in 1.18.2

* **🚀New** Support for Elasticsearch & Kibana 7.2.0
* **🐞Fix** \(ES\) restore indices \("IDX"\) in audit logging
* **🧐Enhancement** \(ES\) New algorithm of setting evaluation order
* **🚀New** \(ES\) JWT claims as dynamic variables. I.e. "@{jwt:claim.json.path}"
* **🚀New** \(ES\) "explode" dynamic variables. I.e. indices: \["@explode{x-indices}"\]
* **🐞Fix** \(PRO/Enterprise\) preserve comments and formatting in YAML editor
* **🐞Fix** \(PRO/Enterprise\) Print error message when session is expired
* **🐞Regression** \(PRO/Enterprise\) Redirect to original link after login
* **🐞Regression** \(PRO/Enterprise\) Broken CSV reporting
* **🧐Enhancement** \(PRO/Enterprise\) Prevent navigating away from YAML editor w/ unsaved changes
* **🐞Fix** \(Enterprise\) Exception when SAML connectors were all disabled
* **🐞Fix** \(Enterprise\) Concurrent tenants could mix up each other kibana index
* **🐞Fix** \(Enterprise\) Cannot inject custom JS if no custom CSS was also declared
* **🐞Fix** \(Enterprise\) Injected JS had no effect on ROR logout button
* **🐞Fix** \(Enterprise\) On narrow screens, the YAML editor showed buttons twice

### What's new in 1.18.1

* **🐞Fix** \(Elasticsearch\) Reindex requests failed for a regression in indices extraction
* **🐞Fix** \(Elasticsearch\) Groups rule erratically failed
* **🐞Fix** \(Elasticsearch\) JWT claims can now contain special characters
* **🧐Enhancement** \(Elasticsearch\) Better ACL History logging
* **🧐Enhancement** \(Elasticsearch\) QueryLogSerializer and old custom log serializers work again
* **🐞Fix** \(PRO/Enterprise\) ReadonlyREST icon in Kibana was white on white
* **🐞Fix** \(Enterprise\) SAML connectors could not be disabled
* **🐞Fix** \(Enterprise\) SAML connector "buttonName" didn't work

### What's new in 1.18.0

* **🚀New** Support for Elasticsearch & Kibana 7.0.1
* **🧐Enhancement** \(Elasticsearch\) empty array values in settings are invalid
* **🐞Security Fix** \(Elasticsearch\) arbitrary x-cluster search referencing local cluster
* **🐞Fix** \(Elasticsearch\) ArrayOutOfBoundException on snapshot operations
* **🧐Enhancement** \(PRO/Enterprise\) History cleaning can now be disabled \("clearSessionOnEvents"\)

### What's new in 1.17.7

* **🚀New** Support for Elasticsearch 7.0.0 \(Kibana is coming soon\)
* **🧐Enhancement** \(Elasticsearch\) rewritten LDAP connector
* **🧐Enhancement** \(Elasticsearch\) new core written in Scala is now GA
* **🐞Fix** \(Enterprise\) devtools requests now honor the currently selected tenancy
* **🐞Security Fix** \(Enterprise/PRO\) Fix "connectorsService" error in installation

### What's new in 1.17.5

* **🚀New** Support for Kibana/Elasticsearch 6.7.1
* **🧐Enhancement** \(Enterprise &gt;= Kibana 6.6.0\) Multiple SAML identity provider
* **🐞Security Fix** \(Enterprise/PRO\) Don't pass auth headers back to the browser
* **🐞Fix** \(Enterprise/PRO\) Missing null check caused error in reporting \(CSV\)
* **🐞Fix** \(Enterprise\) Don't reject requests if SAML groups are not configured
* **🐞Fix** filter/fields rules not working in msearch \(in 6.7.x\)
* **🧐Enhancement** Print whole LDAP search query in debug log

### What's new in 1.17.4

* **🚀New** Support for Kibana/Elasticsearch 6.7.0
* **🧐Enhancement** \(PRO/Enterprise\) JWT query param is the preferred credentials provider
* **🧐Enhancement** \(PRO/Enterprise\) admin users can use indices management
* **🧐Enhancement** \(PRO/Enterprise\) ro users can dismiss telemetry form
* **🐞Fix** Audit logging in 5.1.x now works again
* **🐞Fix** unpredictable behaviour of "filter" and "fields" when using external auth
* **🐞Fix** LDAP ConcurrentModificationException
* **🐞Fix** Audit logging in 5.1.x now works again
* **🐞Fix** \(PRO/Enterprise\) JWT deep-link works again

### What's new in 1.17.3

1.17.2 went unreleased, all changes have been merged in 1.17.3 directly

* **🐞Fix** \(Enterprise\) Tenancy selector showing if user belonged to one group
* **🐞Fix** \(PRO/Enterprise\) RW buttons not hiding for RO users in React Kibana apps
* **🐞Fix** \(Enterprise\) Tenancy templating now works much more reliably
* **🐞Fix** \(Enterprise\) Missing tenancy selector icon after switching tenancy
* **🐞Fix** \(PRO/Enterprise\) barring static files requests caused sudden logout
* **🐞Fix** Numerous fixes to better support Kibana 6.6.x
* **🐞Fix** Critical fixes in new Scala core
* **🐞Fix** Exception in reindex requests caused tenancy templating to fail
* **🧐Enhancement** Bypass cross-cluster search logic if single cluster

### What's new in 1.17.1

* **🐞Fix** \(PRO/Enterprise\) SAML now works well in 6.6.x
* **🐞Fix** \(PRO/Enterprise\) "undefined" authentication error before login
* **🐞Fix** \(Enterprise\) Default space creation failures for new tenants
* **🐞Fix** \(Enterprise\) Icons/titles CSS misalignment in sidebar \(Firefox\)
* **🧐Enhancement**\(Enterprise\) UX: Larger tenancy selector
* **🐞Security Fix** \(Enterprise\) Privilege escalation when changing tenancies under monitoring
* **🐞Fix** \(Elasticsearch\) compatibility fixes to support new Kibana features
* **🧐Enhancements** \(Elasticsearch\) New core and LDAP connector written in Scala is finished, now under QA.

### What's new in 1.17.0

* **🚀New Feature** Support for Kibana/Elasticsearch 6.6.0, 6.6.1
* **🚀New Feature** Internode SSL \(ES 6.3.x onwards\)
* **🧐Enhancement**\(PRO/Enterprise\) UI appearence
* **🧐Enhancement** Made HTTP Connection configurable \(PR \#410\)
* **🐞Fix** slow boot due to SecureRandom waiting for sufficient entropy
* **🐞Fix** Enable kibana\_access:ro to create short urls in es6.3+ \(PR \#408\)

### What's new in 1.16.34

* **🧐Enhancement** X-Forwarded-For header in printed es logs \("XFF"\)
* **🧐Enhancement** kibana_index: ".kibana_@{user}" when user is "John Doe" becomes .kibana\_john\_doe
* **🐞Fix** \(Enteprise\) parse SAML groups from assertion as array of strings
* **🐞Fix** \(Enteprise\) SAMLRequest in location header was URLEncoded twice, broke on some IdP
* **🐞Fix** \(PRO/Enteprise\) "cookiePass" works again, no more need for sticky cookies in load balancers!
* **🐞Fix** \(PRO/Enteprise\) fix redirect loop with JWT deep linking when JWT token expires
* **🧐Enhancement** \(PRO/Enteprise\) fix audit demo page CSS
* **🧐Enhancement** \(Enteprise\) SAML more configuration parameters available
* **🚀New Feature** \(PRO/Enteprise\) set ROR to debug mode \(readonlyrest\_kbn.logLevel: "debug"\)

### What's new in 1.16.33

* **🐞Fix**\(PRO/Enteprise\) compatibility problems with older Kibana versions
* **🐞Fix**\(PRO/Enteprise\) compatibility problems with OSS Kibana version

### What's new in 1.16.32

* **🚀New Feature** "kibanaIndexTemplate": default dashboards and spaces for new tenants
* **🧐Enhancement** Support for ES/Kibana 6.5.4
* **🧐Enhancement** Upgraded LDAP library
* **🧐Enhancement** \(Enterprise\) Now tenants save their CSV exports in their own reporting index
* **🐞Fix**\(PRO/Enteprise\) Support passwords that start and/or end with spaces
* **🐞Fix** \(PRO/Enterprise\) Now reporting works again

### What's new in 1.16.31

* **🧐Enhancement** Support for ES/Kibana 6.5.2, 6.5.3
* **🚧WIP**: Laid out the foundation for LDAP HA support

### What's new in 1.16.29

* **🧐Enhancement** Support for ES/Kibana 6.4.3
* **🚀New Feature** \(PRO/Enterprise\) configurable server side session duration
* **🚀New Feature** \[LDAP\] High Availability: Round Robin or Failover

### What's new in 1.16.28

* **🧐Enhancement** Support for ES/Kibana 6.4.2
* **🐞Fix** \(Enterprise\) Multi tenancy: sometimes changing tenancy would not change kibana index
* **🐞Security Fix** \(Enterprise/PRO\) Avoid echoing Base64 encoded credentials in login form error message
* **🧐Enhancement** \(Enterprise/PRO\) Remove latest search/visualization/dashboard history on logout
* **🧐Enhancement** \(Enterprise/PRO\) Clear transient authentication cookies on login error to avoid authentication deadlocks
* **🐞Fix**: External JWT verification may throw ArrayOutOfBoundException
* **🚧WIP**: Laid out the foundation for internode SSL transport \(port 9300\)

### What's new in 1.16.27

* **🚀New Feature** \[JWT\] external validator: it's now possible to avoid storing the private key in settings
* **🧐Enhancement** Support for ES/Kibana 6.4.1
* **🧐Enhancement** Rewritten big part of ES plugin [documentation](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md)
* **🧐Enhancement** SAML Single log out flow
* **🐞Fix** \(Enterprise/PRO\) [cookiePass](https://github.com/beshu-tech/readonlyrest-docs/blob/master/kibana.md#common-cookie-encryption-secret) works again, but only for Kibana 5.x. Newer Kibana needs sticky sessions in LB.
* **🧐Enhancement** \(Enterprise/PRO\) much faster logout

### What's new in 1.16.26

* **🐞 Fix** \(PRO/Enterprise\) bugs during plugin packaging and installation process

### What's new in 1.16.25

* **🚀New Feature** Users rule: easily restrict external authentication to a list of users
* **🧐Enhancement** Support for ES 5.6.11
* **🐞Hot Fix** \(Enterprise/PRO\) Error 404 when logging in with older versions of Kibana

### What's new in 1.16.24

* **🚀New Feature** \(Enterprise\) SAML Authentication
* **🚀New Feature** Support for Elasticsearch and Kibana 6.4.0
* **🚀New Feature** Headers rule now split in headers\_or and headers\_and
* **🧐Enhancement** Headers rule now allows wildcards
* **🚀New Feature** \(Enterprise\) Multi-tenancy now works also with JSON groups provider
* **🐞 Fix** Multi-tenancy \(Enterprise\) incoherent initial kibana\_index and current group

### What's new in 1.16.23

* **🧐Enhancement** Support for Elastic Stack 6.3.1 and 5.6.10
* **🚀New Feature** \(Enterprise\) Custom CSS injection for Kibana
* **🚀New Feature** \(Enterprise\) Custom Javascript injection for Kibana
* **🚀New Feature** \(PRO/Enterprise\) access paths without need to login \(i.e. /api/status\)
* **🐞Fix** \(PRO/Enterprise\) Navigating to X-Pack APM caused hidden Kibana apps to reappear

### What's new in 1.16.22

* **🚀New Feature:**  map LDAP groups to local groups \(a.k.a. role mapping\)
* **🐞 Fix** \(Elasticsearch\) wildcard aliases resolution not working in "indices" rule.
* **🧐Enhancement:** it is now possible now to use JDK 9 and 10
* **🐞 Fix** \(PRO/Enterprise\) wait forever for login request \(i.e.  slow LDAP servers\)
* **🐞 Fix** \(PRO/Enterprise\) add spinner and block UI if login request is being sent
* **🐞 Fix** \(PRO/Enterprise\) if user is logged out because of LDAP cache expiring + slow authentication, redirect to login.
* **🐞 Fix** \(PRO/Enterprise\) let RO users delete/edit search filters

### What's new in 1.16.21

* **🚀New Feature:** Introducing support for Elasticsearch and Kibana v6.3.0
* **🐞 Fix** \(Enterprise\) multi tenancy - switching tenancy does not always switch kibana index

### What's new in 1.16.20

## ReadonlyREST PRO/Enterprise for Kibana

* **🧐 Enhancement**: when login, forward "elasticsearch.requestHeadersWhitelist" headers. \(useful for "headers" rule  and "proxy\_auth" to work well.\)

## ReadonlyREST for Elasticsearch

* **🚀New Feature**: DLS \(with dynamic variables suppoort\) Thanks [DataSweet](http://www.datasweet.fr/)!
* **🚀 New feature**: Field level security
* **🚀 New rules**: Snapshot, Repositories, Headers
* **🧐 Enhancement**: custom audit serializers: the request content is available
* **🐞 Fix** readonlyrest.yml path discovery
* **🐞 Fix:** LDAP available groups discovery \(tenancy switcher\) corner cases
* **🐞 Fix**: auth\_key\_sha1, auth\_key\_sha256 hashes in settings should be case insensitive
* **🐞 Fix**: LDAP authentication didn't work with local group

