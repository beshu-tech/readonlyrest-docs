# Changelog

### What's new in 1.25.0

* **🚨Security Fix** \(ES\) [Common Vulnerabilities and Exposures \(CVE\)](https://forum.readonlyrest.com/t/update-of-jackson-databind-2-9-6-jar/1728)
* **🚀New** \(ES\) [auth\_key\_pbkdf2 rule](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.25.x/elasticsearch.md#auth_key_pbkdf2)
* **🚀New \(ES\)** [Introduced configuration property defining FLS engine used by fields rule](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.25.x/elasticsearch.md#fields)
* **🧐Enhancement** \(ES\) Fields rule performance improvement
* **🧐Enhancement** \(ES\) Resolved index API support
* **🐞Fix** \(ES\) ["username" in audit entry when user is authenticated via proxy\_auth](https://forum.readonlyrest.com/t/ror-audit-not-logging-user-id)
* **🐞Fix \(ES\)** index resolve action should be treated as readonly action
* **🐞Fix \(ES\)** /\_snapshot and /\_snapshot/\_all should behave the same

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
* **🚀New** \(KBN\) Group names can now be mapped to aliases
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

