# Changelog

### (2026-04-10) What’s new in **ROR 1.69.1**
<details>
<summary><strong>🚨 Security Fix</strong> (KBN) Fixed vulnerability <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-2950">CVE-2026-2950</a></summary>
Fixed a prototype pollution vulnerability (CVE-2026-2950) in the Lodash library used by the Kibana plugin. The issue allowed attackers to bypass a previous fix (CVE-2025-13465) by using array-wrapped path segments in the `_.unset` and `_.omit` functions, potentially enabling deletion of properties from built-in prototypes. The fix upgrades the affected dependency to a patched version.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN) 9.4.0, 9.3.4, 9.3.3, 9.2.8, 8.19.15, 8.19.14 support</summary>
Added compatibility with the latest Kibana versions: 9.4.0, 9.3.4, 9.3.3, 9.2.8, 8.19.15, and 8.19.14. Users running these Kibana releases can now install and use the ReadonlyREST plugin without compatibility issues.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) 9.4.0, 9.3.4, 9.3.3, 9.2.8, 8.19.15, 8.19.14 support</summary>
Added compatibility with the latest Elasticsearch versions: 9.4.0, 9.3.4, 9.3.3, 9.2.8, 8.19.15, and 8.19.14. Users running these Elasticsearch releases can now install and use the ReadonlyREST plugin without compatibility issues.
</details>
<details>
<summary><strong>🚀 New</strong> (ECK) 3.4.0 support</summary>
Added support for Elastic Cloud on Kubernetes (ECK) version 3.4.0, allowing users to deploy and manage ReadonlyREST within ECK-managed Elasticsearch clusters running on Kubernetes.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed <code>jsonwebtoken-ancient</code> being stripped from Kibana builds earlier than 7.11.0</summary>
Fixed an issue where the `jsonwebtoken-ancient` dependency was incorrectly removed during the Kibana plugin build process for versions earlier than 7.11.0, which could cause authentication failures in environments relying on legacy JWT token handling.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Filtered out Fleet-based apps from search results when Management is hidden in Kibana 8.x and 9.x</summary>
Fixed a cosmetic issue where Fleet-based applications (like Fleet, Integrations, etc.) would still appear in Kibana's global search results even when the Management section was hidden by security rules. These apps are now properly filtered out in Kibana 8.x and 9.x.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed <code>/pkp/session-probe</code> requests being blocked by browsers that enforce async-only calls</summary>
Fixed a compatibility issue where session probe requests to the `/pkp/session-probe` endpoint were being blocked by browsers enforcing async-only fetch calls. This ensures the session health check mechanism works correctly across all modern browsers.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed a problem with redirecting to the login form after a 401 error following a session probe check</summary>
Fixed a redirect loop issue where users were not properly redirected to the login form after receiving a 401 error during a session probe check. This ensures a smooth re-authentication flow when the session expires.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) Fixed a missing Kibana access policy in the metadata response when the matched ACL block has no <code>kibana</code> section configured</summary>
Fixed an issue where the Elasticsearch plugin's metadata response was missing the Kibana access policy when the matched ACL block did not have a `kibana` section explicitly configured. This ensures consistent and correct metadata reporting for all ACL configurations.
</details>

### (2026-04-02) What’s new in **ROR 1.69.0**
<details>
<summary><strong>🚨 Security Fix</strong> (KBN) <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-24001">CVE-2026-24001</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-69873">CVE-2025-69873</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-2391">CVE-2026-2391</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-25639">CVE-2026-25639</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-27904">CVE-2026-27904</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-3449">CVE-2026-3449</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-15599">CVE-2025-15599</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-33750">CVE-2026-33750</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2026-4867">CVE-2026-4867</a>, <a href="https://www.tenable.com/cve/CVE-2026-34601">CVE-2026-34601</a>, <a href="https://nvd.nist.gov/vuln/detail/cve-2022-31129">CVE-2022-31129</a></summary>
This release patches 11 CVEs in Kibana's bundled JavaScript dependencies, addressing denial-of-service (ReDoS/infinite loop), XSS, and crash vulnerabilities in libraries such as jsdiff, ajv, qs, axios, minimatch, @tootallnate/once, DOMPurify, brace-expansion, path-to-regexp, xmldom, and moment. All CVEs are fixed by upgrading the affected dependencies to their patched versions.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN/ES) <a href="https://docs.readonlyrest.com/elasticsearch/fleet">Added Fleet support via native API key and service account token authentication (ES 7.14+)</a></summary>
ReadonlyREST now supports Elastic Fleet by validating the two dynamic credential types Fleet creates: service tokens (for Fleet Server) and API keys (for Elastic Agents). The `token_authentication` rule delegates validation to Elasticsearch, so no token values need to be stored in the ROR configuration, and credential rotation requires no config changes.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN/ES) The ReadonlyREST Audit Dashboard available in the Kibana plugin now supports audit events written to data streams</summary>
The ReadonlyREST Audit Dashboard can now visualize audit events stored in data streams, in addition to the previously supported regular indices. This ensures compatibility with modern Elasticsearch deployments that use data streams for time-series audit data.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN/ES) The ReadonlyREST Audit Dashboard provided by the Kibana plugin can now be used with the ECS (Elastic Common Schema) audit index</summary>
The Audit Dashboard now supports the Elastic Common Schema (ECS) format for audit indices, allowing organizations that standardize on ECS to use the dashboard without requiring a custom audit log serializer.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN) <a href="https://forum.readonlyrest.com/t/multi-tenancy-and-link-sharing/1978/3">Added support for opening different tenancies in separate tabs</a></summary>
Users can now open multiple Kibana tenancies in separate browser tabs simultaneously, making it easier to work across different tenants without repeatedly switching contexts.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN) <a href="https://forum.readonlyrest.com/t/multi-tenancy-and-link-sharing/1978/3">Added support for sharing links to Kibana visualizations for the selected tenancy</a></summary>
Visualization links now respect the active tenancy context, enabling users to share direct links to Kibana dashboards and visualizations that automatically open in the correct tenancy for the recipient.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN) Added support for rolling upgrades when upgrading the ROR Elasticsearch plugin and ROR Kibana plugin in a cluster</summary>
Rolling upgrades are now supported for both the ROR Elasticsearch and Kibana plugins, allowing cluster administrators to upgrade nodes one at a time without taking the entire cluster offline.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) Removed the need for manual username input in the impersonation mechanism</summary>
The impersonation feature no longer requires administrators to manually type the target username, streamlining the workflow and reducing the chance of typos when testing user permissions.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) Fixed an error in Kibana caused by empty data streams in Kibana 8.18.0+</summary>
Resolved an error that occurred in Kibana 8.18.0+ when empty data streams were present, ensuring the Kibana UI remains stable and functional regardless of data stream state.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) Added a fallback for an empty <code>indices</code> field in the Audit Dashboard</summary>
The Audit Dashboard now gracefully handles audit events where the `indices` field is empty, preventing visualization errors and ensuring the "Who uses what indices?" view remains functional.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) <a href="https://docs.readonlyrest.com/develop/examples/custom-middleware">Updated custom metadata examples to use the new method. <code>getIdentitySession</code> and <code>getAuthorizationHeaders</code> are now deprecated in favor of <code>getUserRequestIdentity</code>, <code>getIdentitySessionHeaders</code>, and <code>getWhitelistedHeaders</code></a></summary>
The custom middleware API has been updated with new, more clearly named methods. `getIdentitySession` and `getAuthorizationHeaders` are deprecated; users should migrate to `getUserRequestIdentity`, `getIdentitySessionHeaders`, and `getWhitelistedHeaders` for accessing request identity and header information.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch#token_authentication"><code>token_authentication</code> rule extended with <code>api_key</code> and <code>service_token</code> types</a></summary>
The `token_authentication` ACL rule now supports `api_key` and `service_token` as token types, enabling fine-grained access control for Elastic Fleet and other service-to-service authentication scenarios.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) <a href="https://forum.readonlyrest.com/t/distinguish-between-wrong-credentials-and-missing-permissions/2914">Audit log entries and ACL history now include a human-readable reason when a request is denied, making access-control troubleshooting significantly easier</a></summary>
Denied requests now include a clear, human-readable reason in both audit log entries and ACL history, making it much easier to distinguish between authentication failures (wrong credentials) and authorization failures (missing permissions) during troubleshooting.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) Added the new <code>matched_block_names</code> field to audit entries created by audit log serializers other than ECS and custom serializers. The <code>reason</code> field is now deprecated.</summary>
A new `matched_block_names` field has been added to audit entries for non-ECS and non-custom serializers, listing which ACL blocks matched the request. The `reason` field is now deprecated in favor of the more descriptive human-readable reason and `matched_block_names` fields.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) Users defined with LDAP, external, and <code>ror_kbn</code> authentication are no longer treated as local users by the impersonation mechanism</summary>
The impersonation mechanism now correctly distinguishes between local users and users authenticated via LDAP, external providers, or `ror_kbn`. This prevents impersonation from incorrectly applying local-user-only logic to externally managed users.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) The ROR Kibana plugin can no longer be used when the <code>prompt_for_basic_auth: true</code> setting is configured</summary>
When `prompt_for_basic_auth: true` is set in the Elasticsearch plugin configuration, the ROR Kibana plugin will now refuse to operate, preventing an incompatible and insecure configuration where Kibana's session management conflicts with the browser's basic auth prompt.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Resolved a memory leak related to direct calls via the Kibana API</summary>
Fixed a memory leak that occurred when making direct API calls to Kibana, improving long-term stability and preventing gradual memory exhaustion in production environments.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) No longer shows the "Data Set Quality" and "Index management" applications to users with RO or RO_strict access</summary>
The "Data Set Quality" and "Index Management" Kibana applications are now properly hidden from users with read-only (RO) or read-only strict (RO_strict) access, preventing confusion and ensuring the access control model is consistently enforced.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed JWT token authorization when using embedded Kibana</summary>
Resolved an issue where JWT token authorization failed when Kibana was embedded within another application, ensuring seamless SSO integration in embedded Kibana scenarios.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed the styling of the page-not-found screen for Kibana 9.x</summary>
The page-not-found (404) screen now renders with correct styling in Kibana 9.x, eliminating visual glitches and maintaining a polished user experience.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Correctly displays the "Who uses what indices?" Audit Dashboard visualization when indices are not specified in the audit events</summary>
The "Who uses what indices?" visualization in the Audit Dashboard now renders correctly even when audit events lack index information, preventing blank or broken visualizations.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) <a href="https://forum.readonlyrest.com/t/sending-logs-to-another-cluster/2925">Improved stability when sending audit logs to another cluster, so temporary remote cluster outages no longer affect the main cluster</a></summary>
When audit logs are forwarded to a remote Elasticsearch cluster, temporary outages of that remote cluster no longer impact the stability or performance of the main cluster. The audit log shipping is now resilient to connection interruptions.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) Fixed Search Profiler being inactive in Kibana 8.18.0+</summary>
The Search Profiler tool in Kibana 8.18.0+ was not functioning correctly with ROR; this has been fixed, restoring the ability to profile and analyze search query performance.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) <code>beshultd/elasticsearch-readonlyrest</code> images for ES 7.16.x, 7.17.0–7.17.6, and 8.0.x–8.4.x now ship with a patched JDK, replacing bundled JDK 17.0.0–17.0.4 / JDK 18, which crashes on cgroup v2 hosts due to JDK-8287073</summary>
Docker images for the affected Elasticsearch versions now include a patched JDK, resolving crashes on cgroup v2 hosts (common in modern Linux distributions and container runtimes) caused by the JDK-8287073 bug in JDK 17.0.0–17.0.4 and JDK 18.
</details>

### (2026-01-07) What’s new in **ROR 1.68.0**
<details>
<summary><strong>🚨 Security Fix</strong> (KBN) <a href="https://nvd.nist.gov/vuln/detail/CVE-2024-51999">CVE-2024-51999</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-65945">CVE-2025-65945</a></summary>
This release addresses two Kibana-related security vulnerabilities. CVE-2024-51999 was a rejected CVE issued in error and has been removed. CVE-2025-65945 fixes an improper signature verification flaw in the auth0/node-jws library that could allow attackers to bypass HMAC signature verification when using user-provided data in the secret lookup process.
</details>
<details>
<summary><strong>🚨 Security Fix</strong> (ES) <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-67735">CVE-2025-67735</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-66453">CVE-2025-66453</a></summary>
This release patches two Elasticsearch-related security vulnerabilities. CVE-2025-67735 addresses a CRLF injection vulnerability in the Netty framework that could lead to HTTP request smuggling attacks. CVE-2025-66453 fixes a denial-of-service vulnerability in the Rhino JavaScript engine where crafted floating-point numbers could cause excessive CPU consumption.
</details>
<details>
<summary><strong>⚠️Warning</strong> (ES) Audit outputs now use the round-robin strategy for custom audit clusters. <a href="https://docs.readonlyrest.com/elasticsearch/audit#custom-audit-cluster">Audit nodes must belong to the same Elasticsearch cluster; otherwise, audit events may be incomplete</a>  for configuration guidelines.</summary>
The audit system now uses round-robin distribution for custom audit clusters. Administrators must ensure all audit nodes belong to the same Elasticsearch cluster to prevent incomplete audit events. This change improves load distribution but requires proper cluster configuration.
</details>


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀 New** (KBN) 9.3.2, 9.3.1, 9.3.0, 9.2.7, 9.2.6, 9.2.5, 9.2.4, 9.1.10, 8.19.13, 8.19.12, 8.19.11, 8.19.10 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀 New** (ES) 9.3.2, 9.3.1, 9.3.0, 9.2.7, 9.2.6, 9.2.5, 9.2.4, 9.1.10, 8.19.13, 8.19.12, 8.19.11, 8.19.10 support
<details>
<summary><strong>🚀 New</strong> (KBN) Added "Remember last picked tenant" feature for external identity providers</summary>
This feature enhances user experience by remembering the last selected tenant when using external identity providers. Users no longer need to reselect their preferred tenant on each login, streamlining the authentication process for multi-tenant environments.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN) Introduced support for the Kibana Data Set Quality beta application</summary>
ROR now supports the Kibana Data Set Quality beta application, allowing administrators to manage and monitor data quality metrics within their secured Kibana environment. This integration ensures compatibility with Elastic's latest data management tools.
</details>
<details>
<summary><strong>🚀 New</strong> (KBN) Restyled ROR menu featuring searchable tenancy selector</summary>
The ROR menu interface has been redesigned with a modern look and includes a searchable tenancy selector. This improvement makes it easier for users to find and switch between tenants in environments with large numbers of tenants.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) Added new rules: <a href="https://docs.readonlyrest.com/elasticsearch#jwt_authentication"><code>jwt_authentication</code></a> and <a href="https://docs.readonlyrest.com/elasticsearch#jwt_authorization"><code>jwt_authorization</code></a>, as alternatives to the existing <code>jwt_auth</code> rule</summary>
Two new JWT rules provide more granular control over authentication and authorization processes. The `jwt_authentication` rule handles user identity verification, while `jwt_authorization` manages permission assignments, offering greater flexibility compared to the combined `jwt_auth` rule.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch/audit#using-ecs-serializer">New audit log serializer compliant with Elastic Common Schema (ECS)</a></summary>
A new ECS-compliant audit log serializer ensures audit events follow Elastic's standardized format. This improves compatibility with Elastic Stack tools and makes audit data easier to analyze using ECS-aware applications and dashboards.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch/audit#configuration">The audit can be enabled or disabled on the block level</a></summary>
Audit logging can now be controlled at the individual rule block level, providing finer-grained control over what gets logged. Administrators can enable or disable auditing for specific access control blocks while maintaining global audit settings.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) Disabled caching in the Login CSRF protection mechanism.</summary>
Caching has been disabled in the Login CSRF protection to enhance security. This prevents potential CSRF token reuse and ensures each authentication request uses fresh, unique tokens for improved protection against cross-site request forgery attacks.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) Made the tenant indicator always visible and improved its dropdown behavior</summary>
The tenant indicator is now always visible in the UI, providing constant awareness of the current tenant context. The dropdown behavior has been improved for better usability and smoother tenant switching experience.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) Added stack traces to ReadonlyREST KBN plugin error logs for easier debugging</summary>
Error logs now include full stack traces, making it easier for administrators to diagnose and troubleshoot issues. This enhancement significantly improves debugging capabilities by providing detailed error context and call paths.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) <a href="https://forum.readonlyrest.com/t/ldap-connection-timeout-leads-to-authentication-error/2899">Added LDAP connection health checking to prevent stale connection authentication failures</a></summary>
Improved LDAP connection health checking prevents authentication failures caused by stale connections in the pool. This fix addresses issues where daily login attempts would fail after periods of inactivity, particularly in environments with network proxies like Kubernetes.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch/audit#using-configurable-serializer">Enable nested field definitions in the configurable audit log serializer for more flexible audit logging</a></summary>
The configurable audit log serializer now supports nested field definitions, allowing more complex and structured audit data. This provides greater flexibility in customizing audit event formats to match specific organizational requirements.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch/audit#predefined-serializers">The predefined audit log serializers</a> now include a new <code>logged_user</code> field, which contains a human-readable username</summary>
Predefined audit log serializers now include a `logged_user` field displaying human-readable usernames. This enhancement makes audit logs more readable and easier to interpret by showing actual user identities instead of technical identifiers.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Resolved an issue causing the Kibana Search Sessions app to fail on Kibana 8.x</summary>
Fixed a compatibility issue that prevented the Kibana Search Sessions application from functioning properly on Kibana 8.x versions. This ensures full compatibility with Elastic's search session management features.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) <a href="https://forum.readonlyrest.com/t/errors-after-upgrade-kibana-7-17-29-to-8-19-7/2887">Fixed cluster resolution issues that caused Kibana errors and unexpected logouts in versions 8.19.x and above</a></summary>
Resolved cluster resolution problems that were causing Kibana errors and unexpected user logouts after upgrading to Elasticsearch 8.19.x and later versions. This fix addresses compatibility issues introduced in recent Elasticsearch releases.
</details>

### (2025-11-29) What’s new in **ROR 1.67.3**
<details>
<summary><strong>🚀 New</strong> (KBN) 9.2.3, 9.2.2, 9.1.9, 9.1.8, 8.19.9, 8.19.8 support</summary>
ReadonlyREST now officially supports Kibana versions 9.2.3, 9.2.2, 9.1.9, 9.1.8, 8.19.9, and 8.19.8. This ensures compatibility with the latest Kibana security patches and features, allowing administrators to secure their Kibana instances with the most recent releases.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) 9.2.3, 9.2.2, 9.1.9, 9.1.8, 8.19.9, 8.19.8 support</summary>
This release adds official support for Elasticsearch versions 9.2.3, 9.2.2, 9.1.9, 9.1.8, 8.19.9, and 8.19.8. Users can now deploy ReadonlyREST with these Elasticsearch versions to benefit from the latest security updates and performance improvements while maintaining full access control functionality.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) Resolved index resolution compatibility issue with Elasticsearch 9.1.7</summary>
Fixed a compatibility issue where ReadonlyREST had problems resolving index patterns and aliases correctly when running with Elasticsearch 9.1.7. This fix ensures proper index resolution and access control enforcement for users upgrading to or already using Elasticsearch 9.1.7.
</details>

### (2025-11-13) What’s new in **ROR 1.67.2**
<details>
<summary><strong>🚀 New</strong> (KBN) 9.2.1, 9.1.7, 8.19.7 support</summary>
ReadonlyREST now officially supports Kibana versions 9.2.1, 9.1.7, and 8.19.7. This ensures compatibility with the latest Kibana security patches and features, allowing users to upgrade their Kibana deployments while maintaining ReadonlyREST security functionality.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) 9.2.1, 9.1.7, 8.19.7 support</summary>
ReadonlyREST now officially supports Elasticsearch versions 9.2.1, 9.1.7, and 8.19.7. This update provides compatibility with the latest Elasticsearch security updates and performance improvements, ensuring seamless integration of ReadonlyREST security features with these Elasticsearch releases.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed SAML/OIDC provider support behind a reverse proxy when <code>server.rewriteBasePath: false</code> is set in kibana.yml</summary>
This fix resolves an issue where SAML and OpenID Connect authentication providers would fail when Kibana is deployed behind a reverse proxy with `server.rewriteBasePath: false` configuration. The problem occurred because ReadonlyREST was incorrectly handling URL rewriting in this specific deployment scenario, preventing successful authentication through reverse proxy setups.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) Delegated handling of certain internal exceptions to Elasticsearch, preserving native error responses</summary>
This fix improves error handling by allowing Elasticsearch to process certain internal exceptions natively instead of ReadonlyREST intercepting them. This ensures that error responses maintain their original Elasticsearch format and behavior, providing better compatibility with client applications that expect specific error response structures from Elasticsearch.
</details>

### (2025-11-03) What’s new in **ROR 1.67.1**
<details>
<summary><strong>🚀 New</strong> (KBN) 9.2.0, 9.1.6, 8.19.6 support</summary>
Ensures compatibility and full security functionality with the latest Kibana releases, allowing safe upgrades to versions 9.2.0, 9.1.6, and 8.19.6.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) 9.2.0, 9.1.6, 8.19.6 support</summary>
Provides official support for Elasticsearch versions 9.2.0, 9.1.6, and 8.19.6, ensuring the plugin's security features work correctly across the latest stack releases.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (ES) Allow using the <code>actions</code> rule with the <code>kibana</code> rule in the same block when <code>kibana.access: unrestricted</code></summary>
Removes a previous restriction, granting administrators greater configuration flexibility to combine fine-grained action controls with Kibana access rules in unrestricted blocks.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed JWT handling for wrong license edition</summary>
Resolves an authentication failure where JWT validation incorrectly failed based on license type, ensuring reliable JWT authentication regardless of the Elastic license edition.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Suppressed “Forbidden” toast in Discover/Dashboard on Kibana 8.x–9.x</summary>
Eliminates confusing and unnecessary 'Forbidden' pop-up notifications in Kibana's Discover and Dashboard apps when access is correctly denied by ROR rules, improving the user experience.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) <a href="https://forum.readonlyrest.com/t/unable-to-download-reports-from-kibana/2859/2">Resolved report download failure on Kibana 9.1.x</a></summary>
Fixes a critical bug that blocked users from downloading reports (PDF, PNG, CSV) from Kibana 9.1.x dashboards and visualizations, restoring essential reporting functionality.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed timeout when saving Security settings</summary>
Addresses a configuration issue where attempts to save Security settings in Kibana would hang and eventually timeout, preventing administrators from applying critical security changes.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Restored visibility of reports when multiple data streams exist for a reporting index</summary>
Corrects an issue where generated reports became invisible in the Kibana UI if the reporting index was backed by multiple data streams, ensuring all reports are accessible.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed invisible reports for non-tenancy users on Kibana 9.1.x</summary>
Resolves a bug specific to Kibana 9.1.x where users not utilizing multi-tenancy features could not see their generated reports, effectively breaking the reporting interface for them.
</details>

### (2025-10-14) What’s new in **ROR 1.67.0**
<details>
<summary><strong>🚨 Security Fix</strong> (KBN) <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-58754">CVE-2025-58754</a></summary>
This fix addresses CVE-2025-58754, a Denial of Service vulnerability in the Axios HTTP client library (used by the Kibana plugin). When processing `data:` URIs on Node.js, Axios ignored `maxContentLength` and `maxBodyLength` limits, allowing an attacker to trigger unlimited memory allocation and crash the process. The Axios dependency has been updated to a patched version.
</details>
<details>
<summary><strong>🚨 Security Fix</strong> (ES) <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-58057">CVE-2025-58057</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-58056">CVE-2025-58056</a></summary>
Two Netty vulnerabilities have been patched in the Elasticsearch plugin. CVE-2025-58057 is a DoS flaw where the BrotliDecoder and other decompression decoders could allocate unlimited byte buffers, causing Out-of-Memory errors. CVE-2025-58056 is an HTTP request smuggling vulnerability caused by Netty incorrectly accepting standalone newline characters (LF) as chunk-size line terminators instead of requiring CRLF per HTTP/1.1 spec. The Netty dependency has been updated to a fixed version.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch/audit#using-configurable-serializer">Added support for defining a custom audit serializer directly in ROR settings (no code required)</a></summary>
Previously, customizing the format of audit log events required writing a Scala or Java serializer, compiling it into a JAR, and adding it to the plugin classpath. Now you can define a custom audit serializer directly in the ROR configuration using YAML — no coding or compilation needed. This makes it much easier to tailor audit event fields to your specific monitoring and compliance requirements.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch/audit#predefined-serializers">Introduced new predefined audit serializers: <code>ReportingAllEventsAuditLogSerializer</code>, <code>ReportingAllEventsWithQueryAuditLogSerializer</code></a></summary>
Two new built-in audit serializers have been added. `ReportingAllEventsAuditLogSerializer` logs all audit events regardless of verbosity settings, while `ReportingAllEventsWithQueryAuditLogSerializer` does the same but also captures the full request body. These complement the existing serializers and give administrators more granular control over audit logging without needing custom code.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) Added new rules: <a href="https://docs.readonlyrest.com/elasticsearch#ror_kbn_authentication"><code>ror_kbn_authentication</code></a> and <a href="https://docs.readonlyrest.com/elasticsearch#ror_kbn_authorization"><code>ror_kbn_authorization</code></a>, as alternatives to the existing <code>ror_kbn_auth</code> rule</summary>
The existing `ror_kbn_auth` rule combined both authentication and authorization into a single rule. The new `ror_kbn_authentication` and `ror_kbn_authorization` rules allow you to split these concerns into separate ACL blocks, giving you more flexibility to define different authentication methods and authorization logic independently in your security configuration.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) <a href="https://docs.readonlyrest.com/kibana#clock-skew-tolerance">Added OIDC <code>clock-skew-tolerance</code> configuration option in <code>kibana.yml</code></a></summary>
A new `clock-skew-tolerance` configuration option has been added for OIDC authentication in the Kibana plugin. This allows administrators to configure how much time drift (clock skew) is tolerated between the Kibana server and the OIDC identity provider when validating token timestamps, helping to avoid authentication failures in environments with slight clock discrepancies.
</details>
<details>
<summary><strong>🧐 Enhancement</strong> (KBN) <a href="https://docs.readonlyrest.com/kibana#terminate-kibana-on-es-high-watermark">Added option to disable Kibana termination on watermark errors in <code>kibana.yml</code></a></summary>
Previously, when Elasticsearch disk watermark thresholds were exceeded, the ROR Kibana plugin would terminate Kibana to prevent data loss. A new configuration option has been added to `kibana.yml` that allows administrators to disable this automatic termination behavior, giving them more control over how their cluster handles high watermark scenarios.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Logout did not invalidate the app session when the <code>ror_kbn_auth</code> rule was used with local group definitions</summary>
When using the `ror_kbn_auth` rule with locally defined groups, the logout action was not properly invalidating the Kibana application session. This meant that after logging out, the session could potentially remain active. This has been fixed so that logout correctly terminates the session regardless of how groups are defined.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) <a href="https://forum.readonlyrest.com/t/kibana-data-view-filter-not-working-with-keyword/2843">Restored keyword field value suggestions in Discover/Data View filters</a></summary>
After upgrading ROR to versions 1.60+, the Discover and Data View filter dropdowns in Kibana stopped showing value suggestions for keyword fields. This regression has been fixed, restoring the expected autocomplete behavior when filtering by keyword fields in Kibana's data exploration tools.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Integration-based options were visible in search results even when the app was marked as hidden</summary>
When certain Kibana apps were configured as hidden, their integration-based options (such as dashboards or visualizations) could still appear in Kibana's global search results. This fix ensures that when an app is marked as hidden, its associated integration options are also properly excluded from search results.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Index Management appeared in app search results even when the app was declared as hidden</summary>
The Index Management app in Kibana was still appearing in global search results even when administrators had explicitly marked it as hidden in the ROR security configuration. This has been corrected so that hidden apps are fully excluded from search results.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Resolved an issue with CSRF token override when multiple browser tabs were open</summary>
When users had multiple Kibana browser tabs open simultaneously, CSRF token management could cause one tab's token to override another's, leading to unexpected request failures. This issue has been resolved to ensure proper CSRF token isolation across multiple tabs.
</details>
<details>
<summary><strong>🐞 Fix</strong> (KBN) Fixed OIDC compatibility for Kibana 7.10.2 and earlier</summary>
OIDC authentication was broken on older Kibana versions (7.10.2 and earlier) due to compatibility issues with the ROR Kibana plugin. This fix restores proper OIDC support for users running these legacy Kibana versions.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) Restored backward compatibility for custom audit log serializer implementations extending the <code>DefaultAuditLogSerializer</code> class. Custom serializers compiled against ROR 1.65 or 1.66 that use <code>DefaultAuditLogSerializer</code> must be recompiled to work correctly</summary>
Custom audit log serializers that were compiled against ROR 1.65 or 1.66 and extended the `DefaultAuditLogSerializer` class stopped working due to internal API changes. Backward compatibility has been restored, though custom serializers compiled against those versions must be recompiled to work correctly with this release.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) Fixed a defect that broke the "Snapshot and Restore" functionality in Kibana</summary>
A defect in the Elasticsearch plugin was preventing the Snapshot and Restore functionality in Kibana from working correctly. This has been fixed, restoring the ability to create, manage, and restore snapshots through the Kibana UI when ROR security is active.
</details>

### (2025-09-03) What's new in **ROR 1.66.1**
<details>
<summary><strong>🚀 New</strong> (KBN) 9.1.5, 9.1.4, 9.0.8, 9.0.7 8.19.5, 8.19.4, 8.18.7 support</summary>
ReadonlyREST now supports Kibana versions 9.1.5, 9.1.4, 9.0.8, 9.0.7, 8.19.5, 8.19.4, and 8.18.7. This ensures compatibility with the latest Kibana releases and allows users to upgrade their Kibana instances while maintaining ROR security features.
</details>
<details>
<summary><strong>🚀 New</strong> (ES) 9.1.5, 9.1.4, 9.0.8, 9.0.7, 8.19.5, 8.19.4, 8.18.8, 8.18.7 support</summary>
ReadonlyREST now supports Elasticsearch versions 9.1.5, 9.1.4, 9.0.8, 9.0.7, 8.19.5, 8.19.4, 8.18.8, and 8.18.7. This update provides compatibility with the latest Elasticsearch releases across multiple version branches, ensuring users can securely run ROR with current Elasticsearch deployments.
</details>
<details>
<summary><strong>🐞 Fix</strong> (ES) <a href="https://forum.readonlyrest.com/t/ror-1-65-1-java-17/2841">Patching issue in Elasticsearch 9.x, 8.19.x, and 8.18.x that caused startup failures on Java 17</a></summary>
Fixed a compatibility issue that prevented Elasticsearch clusters from starting when using Java 17 with ROR. The patch resolves startup failures affecting Elasticsearch versions 9.x, 8.19.x, and 8.18.x, ensuring smooth operation with modern Java runtime environments.
</details>

### (2025-08-28) What's new in **ROR 1.66.0**
<details>
<summary><strong>🚨Security Fix</strong> (KBN) <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-7339">CVE-2025-7339</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-7783">CVE-2025-7783</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-54419">CVE-2025-54419</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-9288">CVE-2025-9288</a></summary>
🚨Security Fix (KBN) — Patched multiple CVEs affecting Kibana's Node.js dependencies: CVE-2025-7339 (response header manipulation via `on-headers`), CVE-2025-7783 (HTTP Parameter Pollution via `form-data`), CVE-2025-54419 (SAML assertion bypass in Node-SAML), and CVE-2025-9288 (input validation flaw in `sha.js`). These fixes address vulnerabilities ranging from data manipulation to authentication bypass, ensuring your Kibana instances remain secure.
</details>
<details>
<summary><strong>🚨Security Fix</strong> (KBN) <a href="https://forum.readonlyrest.com/t/hidden-functions-are-available-through-the-search/2840/2">Prevented visibility of hidden functions through Kibana UI search</a></summary>
🚨Security Fix (KBN) — Fixed an issue where hidden functions (features restricted by ReadonlyREST rules) could still be discovered and accessed via the Kibana UI search bar. This patch ensures that restricted functionality remains fully hidden from users, closing a potential information disclosure and privilege escalation vector.
</details>
<details>
<summary><strong>🚨Security Fix</strong> (ES) Removed internal failure details from error responses to prevent unintended information disclosure</summary>
🚨Security Fix (ES) — Internal error messages previously exposed stack traces and implementation details in certain failure scenarios. This information is now stripped from error responses, preventing potential leakage of sensitive system internals that could aid an attacker.
</details>
<details>
<summary><strong>🚀New</strong> (KBN) 9.1.3, 9.1.2, 9.0.6, 8.19.3, 8.18.6 support</summary>
🚀New (KBN) — Added compatibility with Kibana versions 9.1.3, 9.1.2, 9.0.6, 8.19.3, and 8.18.6. Users on these versions can now install and run ReadonlyREST Kibana plugin without compatibility issues.
</details>
<details>
<summary><strong>🚀New</strong> (ES) 9.1.3, 9.1.2, 9.0.6, 8.19.3, 8.18.6 support</summary>
🚀New (ES) — Added compatibility with Elasticsearch versions 9.1.3, 9.1.2, 9.0.6, 8.19.3, and 8.18.6. The Elasticsearch plugin now fully supports these releases.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (ES) Refined user metadata selection logic during login to prioritize matched blocks associated with a defined Kibana index</summary>
🧐Enhancement (ES) — Improved the login flow so that when multiple ACL blocks match a user, the system now prioritizes the block that is associated with a defined Kibana index. This results in more predictable and correct user metadata assignment, especially in multi-block configurations.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (ES) Patching: improved handling of the consent flag when provided via environment variables for more reliable configuration</summary>
🧐Enhancement (ES) — Enhanced the patching mechanism to more reliably process the consent flag when it is supplied through environment variables. This reduces configuration errors and ensures smoother automated deployments.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Resolved issue with index deletion in <strong>Index Management</strong> via Kibana UI</summary>
🐞Fix (KBN) — Fixed a bug where users with appropriate permissions were unable to delete indices through the Kibana Index Management interface. Index deletion now works correctly when authorized by ReadonlyREST rules.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Corrected document display in <strong>Discover</strong> when indices are defined in the user ACL block</summary>
🐞Fix (KBN) — Resolved an issue where documents were not displayed correctly in the Kibana Discover section when indices were explicitly defined in the user's ACL block. Document browsing now works as expected in restricted index configurations.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Fixed an error preventing <strong>Spaces</strong> from being deleted in Kibana <strong>9.1.0</strong></summary>
🐞Fix (KBN) — Addressed a specific error that prevented users from deleting Kibana Spaces in version 9.1.0 when ReadonlyREST was active. Space management now functions correctly on this version.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Corrected handling of <code>readonlyrest_kbn.whitelistedPaths</code> in <code>kibana.yml</code> when <code>xpack.security.enabled: true</code></summary>
🐞Fix (KBN) — Fixed a configuration handling issue where the `readonlyrest_kbn.whitelistedPaths` setting in `kibana.yml` was not properly respected when `xpack.security.enabled` was set to `true`. Whitelisted paths now work reliably regardless of the xpack security setting.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Resolved startup issues for Kibana versions <strong>7.9.0 → 7.10.2</strong></summary>
🐞Fix (KBN) — Fixed a compatibility regression that caused ReadonlyREST to fail during Kibana startup on versions 7.9.0 through 7.10.2. Users on these older Kibana releases can now run the plugin without startup errors.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Fixed report generation when <code>xpack.security.enabled: true</code> and <code>xpack.encryptedSavedObjects.encryptionKey</code> is set in Kibana <strong>8.19.x</strong> and <strong>9.1.x</strong></summary>
🐞Fix (KBN) — Resolved an issue where report generation (e.g., PDF/CSV exports) would fail in Kibana 8.19.x and 9.1.x when both `xpack.security.enabled` and `xpack.encryptedSavedObjects.encryptionKey` were configured. Reports now generate successfully in these environments.
</details>

### (2025-07-15) What's new in **ROR 1.65.1**
<details>
<summary><strong>🚀New</strong> (KBN) 9.1.1, 9.1.0, 9.0.5, 9.0.4, 8.19.2, 8.19.1, 8.19.0, 8.18.5, 8.18.4, 8.17.10, 8.17.9 support</summary>
ReadonlyREST now supports the latest Kibana versions including 9.1.1, 9.1.0, 9.0.5, 9.0.4, 8.19.2, 8.19.1, 8.19.0, 8.18.5, 8.18.4, 8.17.10, and 8.17.9. This ensures compatibility with recent Kibana releases and their security patches, allowing users to upgrade their Kibana instances while maintaining ReadonlyREST security features.
</details>
<details>
<summary><strong>🚀New</strong> (ES) 9.1.1, 9.1.0, 9.0.5, 9.0.4, 8.19.2, 8.19.1, 8.19.0, 8.18.5, 8.18.4, 8.17.10, 8.17.9 support</summary>
The plugin now supports Elasticsearch versions 9.1.1, 9.1.0, 9.0.5, 9.0.4, 8.19.2, 8.19.1, 8.19.0, 8.18.5, 8.18.4, 8.17.10, and 8.17.9. This update provides compatibility with the latest Elasticsearch releases, including security updates and performance improvements from Elastic.
</details>
<details>
<summary><strong>🚀New</strong> (ECK) 3.1.0 support</summary>
ReadonlyREST now supports Elastic Cloud on Kubernetes (ECK) version 3.1.0. This enables users running Elasticsearch on Kubernetes through ECK to leverage ReadonlyREST's security features in their containerized environments with the latest ECK release.
</details>
<details>
<summary><strong>🐞Fix</strong> (ES) Docker images now start correctly when <code>I_UNDERSTAND_AND_ACCEPT_ES_PATCHING</code> is set.</summary>
Fixed an issue where Elasticsearch Docker images with ReadonlyREST would fail to start when the environment variable `I_UNDERSTAND_AND_ACCEPT_ES_PATCHING` was set. This variable is commonly used in Elasticsearch Docker deployments to acknowledge patching terms, and the fix ensures smooth container startup.
</details>

### (2025-07-10) What's new in **ROR 1.65.0**
<details>
<summary><strong>🚨Security Fix</strong> (KBN) <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-5889">CVE-2025-5889</a></summary>
A vulnerability in the `brace-expansion` library (up to versions 1.1.11, 2.0.1, 3.0.0, 4.0.0) could lead to inefficient regular expression complexity and potential denial of service. This fix addresses the issue by updating the affected dependency to a patched version.
</details>
<details>
<summary><strong>🚨Security Fix</strong> (ES) <a href="https://nvd.nist.gov/vuln/detail/cve-2024-29857">CVE-2024-29857</a> (when FIPS SSL is used)</summary>
A high-severity vulnerability (CVSS 7.5) in the Bouncy Castle cryptographic library (before 1.78) could cause excessive CPU consumption and denial of service when importing an EC certificate with specially crafted F2m parameters. This fix updates the Bouncy Castle dependency and is relevant when FIPS-compliant SSL is configured.
</details>
<details>
<summary><strong>🚀New</strong> (KBN)  Added support for configuring <a href="https://www.elastic.co/docs/troubleshoot/kibana/using-kibana-server-logs">JSON log format</a> in <code>kibana.yml</code>.</summary>
Administrators can now enable structured JSON logging for Kibana, making it easier to parse, index, and analyze logs with centralized log management tools like Elasticsearch and Logstash.
</details>
<details>
<summary><strong>🚀New</strong> (ES) <a href="https://docs.readonlyrest.com/elasticsearch/audit#configuration">Added support for a new output type: <code>data_stream</code> in audit logging</a>.</summary>
Audit events can now be stored in Elasticsearch data streams instead of regular indices. Data streams offer better lifecycle management, automatic rollover, and simplified retention policies via ILM. If the specified data stream doesn't exist, ReadonlyREST creates it automatically along with the necessary component templates and index template.
</details>
<details>
<summary><strong>🚀New</strong> (ES) Included Elasticsearch node name and cluster name in the audit reports.</summary>
Audit log entries now contain the originating Elasticsearch node name and cluster name, providing better traceability and context when auditing requests across multi-node or multi-cluster deployments.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (KBN) Logged detailed messages when the CSRF token has expired.</summary>
Improved logging now provides clearer, more descriptive messages when a CSRF token expires, helping administrators diagnose and troubleshoot authentication-related issues more efficiently.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (KBN) <a href="https://docs.readonlyrest.com/kibana#user-info-source-methods">Added <code>id_token</code> as a valid option for <code>userInfoSource</code></a>.</summary>
Administrators can now configure the `userInfoSource` setting to use the `id_token` directly as the source of user information, providing more flexibility in OIDC-based authentication workflows.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (ES) Improved handling of JVM properties related to ROR settings.</summary>
The way ReadonlyREST processes and applies JVM property-based configuration settings has been refined, resulting in more reliable behavior and better error handling when custom JVM options are used.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Fixed OIDC logout redirection issue by switching <code>redirect_uri</code> to <code>id_token_hint</code> and using <code>post_logout_redirect_uri</code>.</summary>
The OIDC logout flow has been corrected to use the standard `id_token_hint` parameter instead of `redirect_uri`, along with proper `post_logout_redirect_uri` handling, ensuring users are correctly redirected after logout.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) The ReadonlyREST Kibana plugin now accepts custom appender names defined in <code>kibana.yml</code>.</summary>
Previously, the plugin would reject custom logging appender names configured in Kibana's logging configuration. This fix ensures compatibility with custom appender setups.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) When "Remember Group After Logout" is enabled, groups without access are correctly ignored during login.</summary>
Fixed a bug where previously remembered groups that no longer had access permissions could still be applied during re-authentication. Now only groups with valid access are considered.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Fixed issue where the Kibana index template was not applied for Kibana versions ≥ 8.8.0.</summary>
A compatibility issue with Kibana 8.8.0 and newer prevented the ROR index template from being properly applied. This fix restores correct template application for these versions.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Resolved a bug with <code>readonlyrest_kbn.resetKibanaIndexToTemplate: true</code> for Kibana 7.x.</summary>
The index reset functionality, which restores the Kibana index to match the expected template, was not working correctly on Kibana 7.x. This fix ensures the setting behaves as intended.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Fixed an issue where a custom session index name was not respected after Kibana restart.</summary>
When a custom session index name was configured, Kibana would revert to the default session index after a restart. This fix ensures the custom session index name is persisted and used correctly across restarts.
</details>
<details>
<summary><strong>🐞Fix</strong> (ES) Fixed an issue preventing snapshots from being restored when no indices were specified.</summary>
Restoring a snapshot without explicitly specifying indices (i.e., restoring all indices) was failing under certain conditions. This fix ensures that snapshot restore operations work correctly even when no index list is provided.
</details>
<details>
<summary><strong>🐞Fix</strong> (ES) File ownership and permissions are now preserved during <code>ror-tools</code> patch and unpatch operations.</summary>
Previously, running `ror-tools` to patch or unpatch Elasticsearch could alter file ownership and permissions. This fix ensures that the original file attributes are maintained throughout the patching process.
</details>

### (2025-05-17) What's new in **ROR 1.64.2**
<details>
<summary><strong>🚀New</strong> (KBN) 9.0.3, 9.0.2, 8.18.3, 8.18.2, 8.17.8, 8.17.7, 7.17.29 support</summary>
ReadonlyREST now supports the latest Kibana versions including 9.0.3, 9.0.2, 8.18.3, 8.18.2, 8.17.8, 8.17.7, and 7.17.29. This ensures compatibility with recent Kibana releases and their security updates.
</details>
<details>
<summary><strong>🚀New</strong> (ES) 9.0.3, 9.0.2, 8.18.3, 8.18.2, 8.17.8, 8.17.7, 7.17.29 support</summary>
ReadonlyREST now supports the latest Elasticsearch versions including 9.0.3, 9.0.2, 8.18.3, 8.18.2, 8.17.8, 8.17.7, and 7.17.29. This provides compatibility with recent Elasticsearch releases and their security patches.
</details>
<details>
<summary><strong>🐞Fix</strong> (ES) <a href="https://forum.readonlyrest.com/t/ror-1-64-0-for-es9-0-1-windows-setup/2778">Fixed an issue with Elasticsearch patching process on Windows operating systems</a></summary>
Resolved a Windows-specific error that occurred during the Elasticsearch patching process with ReadonlyREST. The issue was successfully reproduced by the support team and fixed to ensure smooth installation on Windows environments.
</details>

### (2025-05-13) What's new in **ROR 1.64.1**
<details>
<summary><strong>🐞Fix</strong> (ES) Correct patching verification in ROR Docker image entrypoint</summary>
This fix addresses an issue in the Docker image entrypoint script where patching verification was not functioning correctly. The entrypoint script, which handles the application of security patches and configuration updates, now properly validates that patches are applied successfully before proceeding with container startup, ensuring reliable deployment of ROR-secured Elasticsearch instances.
</details>

### (2025-05-11) What's new in **ROR 1.64.0**
<details>
<summary><strong>🚨Security Fix</strong> (KBN) <a href="https://nvd.nist.gov/vuln/detail/CVE-2024-53382">CVE-2024-53382</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-27789">CVE-2025-27789</a>, <a href="https://www.cve.org/CVERecord?id=CVE-2025-29774">CVE-2025-29774</a></summary>
This release addresses three security vulnerabilities in Kibana dependencies: CVE-2024-53382 is a DOM clobbering XSS vulnerability in PrismJS syntax highlighter (versions ≤1.29.0), CVE-2025-27789 is a performance/DoS issue in Babel's regex polyfill with quadratic complexity, and CVE-2025-29774 (details not fully available). These fixes prevent potential cross-site scripting attacks and denial of service scenarios.
</details>
<details>
<summary><strong>🚨Security Fix</strong> (ES) <a href="https://nvd.nist.gov/vuln/detail/CVE-2023-3894">CVE-2023-3894</a>, <a href="https://nvd.nist.gov/vuln/detail/CVE-2025-25193">CVE-2025-25193</a></summary>
This release patches two Elasticsearch-related vulnerabilities: CVE-2023-3894 is a Denial of Service vulnerability in jackson-dataformats-text library (versions <2.15.0) that could cause stack overflow when parsing malicious TOML data, and CVE-2025-25193 is a Windows-specific DoS vulnerability in Netty (versions ≤4.1.118.Final) where large environment files could crash the application. These fixes enhance system stability and security.
</details>
<details>
<summary><strong>⚠️Warning</strong> (ES) Acknowledgement needs to be accepted before the Elasticsearch patching process. For scripts, you can <a href="https://docs.readonlyrest.com/elasticsearch#id-3.-patch-elasticsearch">set the flag</a> to automate the process.</summary>
When patching Elasticsearch for ReadonlyREST installation, users must now explicitly acknowledge the patching process. For automated deployments, administrators can set a configuration flag to bypass the manual acknowledgement, enabling script-based automation of the patching workflow.
</details>
<details>
<summary><strong>🚀New</strong> (KBN) Added an endpoint to retrieve all user tenancies via the ReadonlyREST API. See the <a href="https://portal.readonlyrest.com/docs/swagger/master#/User's%20tenants/get_api_ror_user_tenants">ReadonlyREST API Documentation</a> for usage details.</summary>
A new API endpoint has been added to retrieve all tenancies associated with a user. This enables programmatic access to multi-tenancy information, allowing administrators and applications to query and manage user tenancy assignments through the ReadonlyREST API interface.
</details>
<details>
<summary><strong>🚀New</strong> (KBN) Introduced support for passing <code>x-ror-tenancy-id</code> in direct Kibana requests. See the <a href="https://portal.readonlyrest.com/docs/swagger/master#/Example%20ReadonlyREST%20headers%20usage%20with%20Kibana%20API/get_api__">ReadonlyREST API Documentation</a> for details.</summary>
Direct Kibana API requests can now include the `x-ror-tenancy-id` header to specify the target tenancy context. This allows applications and scripts to make requests within specific tenancy contexts without relying on session-based tenancy selection, improving automation and integration capabilities.
</details>
<details>
<summary><strong>🚀New</strong> (KBN) Introduced support for passing <code>x-ror-impersonating</code> in direct Kibana requests. See the <a href="https://portal.readonlyrest.com/docs/swagger/master#/Example%20ReadonlyREST%20headers%20usage%20with%20Kibana%20API/get_api__">ReadonlyREST API Documentation</a> for details.</summary>
The new `x-ror-impersonating` header enables administrators to make Kibana API requests on behalf of other users. This feature supports administrative workflows where privileged users need to perform actions or troubleshoot issues within another user's security context while maintaining audit trails.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (KBN) Retains the currently selected group information after user logout. This setting is user-configurable and disabled by default.</summary>
Kibana now optionally preserves the user's selected group/tenancy information across logout/login cycles. This user-preference setting (disabled by default) improves user experience by maintaining context between sessions, reducing the need to reselect groups upon each login.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (KBN) Displays <a href="https://docs.readonlyrest.com/elasticsearch#unauthorized-response-configuration">detailed "reason" messages from the ROR Elasticsearch</a> response in the login form instead of a generic "Wrong credentials" message.</summary>
Login failures now show specific error messages from Elasticsearch's ReadonlyREST plugin rather than generic "Wrong credentials" messages. This provides users with actionable feedback about authentication issues, such as account lockouts, expired credentials, or specific authorization failures.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (KBN) Added support for passing additional <a href="https://docs.readonlyrest.com/kibana#additional-parameters">SAML</a> and <a href="https://docs.readonlyrest.com/kibana#additional-parameters">OIDC</a> config parameters via <code>kibana.yml</code>.</summary>
Extended configuration options for SAML and OIDC authentication providers can now be specified directly in kibana.yml. This allows administrators to customize authentication flows with provider-specific parameters without modifying plugin code, enhancing integration flexibility with enterprise identity systems.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (KBN) Adjusted ReadonlyREST plugin UI styles for compatibility with Kibana 9.x.</summary>
The ReadonlyREST plugin interface has been updated with CSS and styling adjustments to ensure proper display and functionality within Kibana 9.x environments. This maintains visual consistency and usability as Kibana evolves its user interface framework.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (ES) Username duplication check in the "users" section of ROR ES settings can <a href="https://docs.readonlyrest.com/elasticsearch#users_section_duplicate_usernames_detection">be optionally disabled</a>.</summary>
Administrators can now optionally disable the duplicate username validation in Elasticsearch settings. This provides flexibility for complex deployment scenarios where username duplication might be intentional or managed through external systems, while maintaining the default validation for security.
</details>
<details>
<summary><strong>🧐Enhancement</strong> (ES) Added support for <a href="https://docs.readonlyrest.com/elasticsearch#global-settings"><code>readonlyrest.global_settings</code></a> in Elasticsearch ROR settings.</summary>
Elasticsearch configuration now supports `readonlyrest.global_settings` for centralized management of plugin-wide parameters. This enables consistent configuration across clusters and simplifies administration by separating global settings from rule-specific configurations.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Resolved an unhandled error when <code>logging.root.level</code> is set to <code>all</code> in <code>kibana.yml</code>.</summary>
Fixed a crash that occurred when Kibana's logging.root.level was configured as "all" in kibana.yml. The plugin now properly handles this logging configuration, preventing startup failures and ensuring compatibility with verbose logging settings for debugging purposes.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Fixed an issue with retrieving username and group information in AFDS OIDC.</summary>
Corrected a bug where Azure AD Federated Services (AFDS) OIDC authentication failed to properly extract username and group information from identity tokens. This fix ensures proper user identification and group-based authorization for Azure AD-integrated deployments.
</details>
<details>
<summary><strong>🐞Fix</strong> (KBN) Fixed an issue with passing <code>x-ror-correlation-id</code> to the ReadonlyREST API request.</summary>
Resolved a problem where the `x-ror-correlation-id` header was not being properly passed through to ReadonlyREST API requests. This fix ensures correlation IDs are correctly transmitted for request tracing, debugging, and audit logging across the authentication and authorization pipeline.
</details>

### (2025-03-12) What's new in **ROR 1.63.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (KBN) [CVE-2025-26791](https://www.cve.org/CVERecord?id=CVE-2025-26791), [CWE-772](https://cwe.mitre.org/data/definitions/772.html)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2024-57699](https://nvd.nist.gov/vuln/detail/CVE-2024-53990) [CVE-2025-25193](https://nvd.nist.gov/vuln/detail/CVE-2025-25193) [CVE-2025-24970](https://nvd.nist.gov/vuln/detail/CVE-2025-24970)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 9.0.1, 9.0.0, 9.0.0-rc1, 9.0.0-beta1, 8.18.1, 8.18.0, 8.17.6, 8.17.5, 8.17.4, 8.16.6 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 9.0.1, 9.0.0, 9.0.0-rc1, 9.0.0-beta1, 8.18.1, 8.18.0, 8.17.6, 8.17.5, 8.17.4, 8.16.6 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [Added `groups_not_any_of` and `groups_not_all_of` rules](https://forum.readonlyrest.com/t/support-kbn-ent-managing-forbidden-messages/2623)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [New unified and simplified syntax for groups rules](https://docs.readonlyrest.com/elasticsearch#groups-rules)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) For Kibana >= 8.14.0: Added backward compatibility to hide the Dashboard app by declaring Analytics|Dashboard and Analytics|Dashboards in the `kibana.hide_apps` rule


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Added information about skipping patching confirmation prompt to the patching helper


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) [When Kibana is opened in multiple browser tabs, logging into Kibana in one tab automatically logs in all browser tabs]


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Don't terminate Kibana when disk reaches low watermark


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) For Kibana >= 8.15.0: Added support for reporting data stream multitenancy


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Silenced "Error fetching fields for index pattern" toast messages due to forbidden response in Kibana Dashboard and Discover page


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) For Kibana >= 8.17.0: Fixed Elasticsearch navigation header being visible when `kibana.hide_apps: [ "Elasticsearch" ]`


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [For Kibana >= 8.5.0: Fixed Dev tools play buttons not being visible for RO users](https://forum.readonlyrest.com/t/ldap-multitenancy-with-no-group-name-to-index-name-relation/2742/8)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fixed an issue with hiding the dashboard app when using regular expressions in the kibana_hide_apps field


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Fixed various issues with restoring snapshot API


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Fixed data streams, index, and component templates being forbidden for RW users in stack management

### (2025-01-24) What's new in **ROR 1.62.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2024-53990](https://nvd.nist.gov/vuln/detail/CVE-2024-53990)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (KBN) [CVE-2024-21538](https://www.cve.org/CVERecord?id=CVE-2024-21538), [CVE-2024-47764](https://www.cve.org/CVERecord?id=CVE-2024-47764), [CVE-2024-52798](https://www.cve.org/CVERecord?id=CVE-2024-52798)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**⚠️Warning** (KBN) Updated [`readonlyrest_kbn: license: activationKeyRefreshInterval`](https://forum.readonlyrest.com/t/restricting-access-to-some-spaces/2633/4) - the maximum refresh interval is now set to 1 day.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES|KBN) Introduced support for [Elastic APM (Application Performance Monitoring)](https://www.elastic.co/observability/application-performance-monitoring).


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.17.3, 8.17.2, 8.17.1, 8.16.5, 8.16.4, 8.16.3, 7.17.28 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.17.3, 8.17.2, 8.17.1, 8.16.5, 8.16.4, 8.16.3, 7.17.28 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Added [Kibana images with the preinstalled ReadonlyREST plugin for the arm64 platform](https://hub.docker.com/r/beshultd/kibana-readonlyrest) on Docker Hub.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) Added [Elasticsearch images with the preinstalled ReadonlyREST plugin for the arm64 platform](https://hub.docker.com/r/beshultd/elasticsearch-readonlyrest) on Docker Hub.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) [Introduced validation to prevent multiple username entries in the users section.](https://forum.readonlyrest.com/t/ror-1-57-3-es-8-13-2-double-usernames-allowed/2621/2)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [Resolved an issue with exit patching-based commands.](https://forum.readonlyrest.com/t/restricting-access-to-some-spaces/2633/6)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Addressed a bug in Kibana 8.16.0 and later versions to hide the permissions tab in a space.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fixed a compatibility issue where OIDC and SAML didn't work in Kibana versions earlier than 7.11.0.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Ensured user settings are overridden only for the default space.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Relaxed restrictions on snapshot restoration during index checks.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Resolved issue with Stack Monitoring access when `xpack.security.enabled: true` is configured.

### (2024-11-20) What's new in **ROR 1.61.1**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [Data leak through the ESQL API](https://forum.readonlyrest.com/t/eql-requests-returns-data-even-though-they-aren-t-allowed/2679) (for ES >= 8.11.0)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (KBN) [CVE-2024-21538](https://www.cve.org/CVERecord?id=CVE-2024-21538), [CVE-2024-47764](https://www.cve.org/CVERecord?id=CVE-2024-47764)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2024-47535](https://nvd.nist.gov/vuln/detail/CVE-2024-47535)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.17.0, 8.16.2, 8.16.1, 8.16.0, 8.15.5, 7.17.27, 7.17.26 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.17.0, 8.16.2, 8.16.1, 8.15.5, 7.17.27, 7.17.26 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) ESQL support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Elasticsearch red status shouldn't kill the Kibana process on initialization

### (2024-11-12) What's new in **ROR 1.61.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (KBN) [CVE-2024-47764](https://www.cve.org/CVERecord?id=CVE-2024-47764)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**⚠️Warning** (KBN) Acknowledgement needs to be accepted before a Kibana patching process. For scripts, you can [set a flag](https://docs.readonlyrest.com/kibana#patching-kibana) to automate a process (edited)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.15.4 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.16.0, 8.15.4 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) There is an option to define [a custom response for users in ACL block with the 'forbid' policy](https://docs.readonlyrest.com/elasticsearch#unauthorized-response-configuration)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Set-Cookie is not returned with KBN API response


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Reduce the amount of ReadonlyREST session updates


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Kibana plugin won't start until the connection with Elasticsearch is established


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) API and activation key tabs in the Security settings are visible only for the admin or unrestricted access users


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) detecting issues related to high disk watermark warning


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) License expiration info only for admin and unrestricted access users


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) index exclusion (dash) syntax support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Don't stop Kibana when correlationId is not available in the session


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Provide additional [SAML configuration options](https://docs.readonlyrest.com/kibana#usage-with-active-directory-federation-services) to handle Active Directory Federation Services (ADFS) properly


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) login page customization should be a PRO feature instead of an Enterprise


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Logging to file doesn't work for Kibana 8.x


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Snapshot Status API - forbidden response while checking the status of all snapshots of the given repository


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Snapshot API - misc issues for ES 6.x

### (2024-09-15) What's new in **ROR 1.60.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.15.3, 8.15.2, 7.17.25 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.15.3, 8.15.2, 7.17.25 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN|ES) [ECK support documentation](https://docs.readonlyrest.com/eck)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) configurable ROR YAML settings max size


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**⚠️Warning** (ES) The prompt for basic authorization is disabled by default. To keep the previous behavior, set `readonlyrest.prompt_for_basic_auth` to `true` in the ROR configuration


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) There is an option to define [client authentication methods](https://docs.readonlyrest.com/kibana#client-authentication-methods) in the `kibana.yml` via `readonlyrest_kbn.auth.<YOUR_OIDC_CONFIG>.tokenEndpointAuthMethod`, 'client_secret_post' or ''client_secret_basic'


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Stop Kibana when enabled features are not available


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) HTTP 400 (bad request) issue when there is a Nginx proxy server between es and Kibana


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fix for the problem with correctly hiding Management features `ROR Manage Kibana` defined in the readonlyrest.yml `kibana_hide_apps` property


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) ROR KBN docker image: passing ROR settings as ENVs fixes


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Data stream backing indices access issue with the indices rule](https://forum.readonlyrest.com/t/requested-index-doesnt-exist/2573)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Fix for the problem with remote access to data stream aliases](https://forum.readonlyrest.com/t/requested-index-doesnt-exist/2573)

### (2024-08-01) What's new in **ROR 1.59.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.15.1, 8.15.0, 7.17.24, 7.17.23, 6.7.x support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.15.1, 8.15.0, 7.17.24, 7.17.23 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Replace a broken Alert and Connectors applications with the link to our [new tool](https://anaphora.it) for Reports and alerting for Kibana > 8.6.0 (edited)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Handling reporting URL for report generation


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Embedding with inline JWT is a feature available only in ReadonlyREST PRO and Enterprise


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Patcher `UnsupportedOperationException` issue on Windows](https://forum.readonlyrest.com/t/ror-1-58-0-for-es8-14-3-windows-setup/2577)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) for the problem with `_async_search` on ES 8.14.x

### (2024-06-30) What's new in **ROR 1.58.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix**(KBN) [CVE-2022-39353](https://www.cve.org/CVERecord?id=CVE-2022-39353), [CVE-2020-7753](https://www.cve.org/CVERecord?id=CVE-2020-7753), [CVE-2022-37616](https://www.cve.org/CVERecord?id=CVE-2022-37616), [CVE-2024-29041](https://www.cve.org/CVERecord?id=CVE-2024-29041), [CVE-2022-0691](https://www.cve.org/CVERecord?id=CVE-2022-0691), [CVE-2021-3801](https://www.cve.org/CVERecord?id=CVE-2021-3801), [CVE-2022-25883](https://www.cve.org/CVERecord?id=CVE-2022-25883), [CVE-2022-0512](https://www.cve.org/CVERecord?id=CVE-2022-0512), [CVE-2022-0686](https://www.cve.org/CVERecord?id=CVE-2022-0686), [CVE-2022-0639](https://www.cve.org/CVERecord?id=CVE-2022-0639), [CVE-2022-25881](https://www.cve.org/CVERecord?id=CVE-2022-25881), [CVE-2023-0842](https://www.cve.org/CVERecord?id=CVE-2023-0842), [CVE-2017-16137](https://www.cve.org/CVERecord?id=CVE-2017-16137), [CVE-2022-33987](https://www.cve.org/CVERecord?id=CVE-2022-33987), [CVE-2022-23647](https://www.cve.org/CVERecord?id=CVE-2022-23647), [CVE-2022-36083](https://www.cve.org/CVERecord?id=CVE-2022-36083), [CVE-2024-28176](https://www.cve.org/CVERecord?id=CVE-2024-28176)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) [Kibana images with preinstalled ReadonlyREST plugin in Docker Hub](https://hub.docker.com/r/beshultd/kibana-readonlyrest)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.14.3, 8.14.2 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.14.3, 8.14.2 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) ["structured groups" feature](https://github.com/beshu-tech/readonlyrest-docs/blob/develop/details/structured-groups.md) (authorization rules group names and group IDs can be defined separately)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) New `readonlyrest_kbn.cookies.secure` and `readonlyrest_kbn.cookies.sameSite` cookie settings via kibana.yml


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) improved error logging on the creation of LDAP connectors


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) Patcher - invalid state after patching detection improvements


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Impersonation and session probe logout issue


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [Problem with the number of replicas and index template, where the number of replicas was always set to 1. Now, the default value will be the same, as in the case of the Kibana index](https://forum.readonlyrest.com/t/0-replicas-for-single-node-clusters/2530)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fix problem with multi-tenancy features when xpack.security.enabled: true

### (2024-05-18) What's new in **ROR 1.57.3**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2024-34447](https://nvd.nist.gov/vuln/detail/CVE-2024-34447)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.14.1, 8.14.0, 7.17.22 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.14.1, 8.14.0, 7.17.22 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) The CSRF cookie name issue that caused the "Wrong credentials" error during login


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Automatic migration issue for Kibana >= 8.8.0 that caused the "mapping set to strict, dynamic introduction of... error

### (2024-05-05) What's new in **ROR 1.57.2**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.13.4, 8.13.3, 7.17.21 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.13.4, 8.13.3, 7.17.21 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Kibana <= 7.2.1 doesn't run


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Provides a way to migrate an existing session index to the new session


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Patching issue for Elasticsearch installed from packages](https://forum.readonlyrest.com/t/bootstrap-error-es/2574)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Patching issue for Elasticsearch OSS versions

### (2024-04-29) What's new in **ROR 1.57.1**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) configuration parsing regression: one group definition can be a string

### (2024-04-28) What's new in **ROR 1.57.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2024-29025](https://nvd.nist.gov/vuln/detail/CVE-2024-29025)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [LDAP Connector](https://docs.readonlyrest.com/elasticsearch#configuration-notes) feature: groups server-side filtering


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [LDAP Connector](https://docs.readonlyrest.com/elasticsearch#configuration-notes) feature: skip user search option when user attribute is `cn`


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**⚠️Warning** (KBN|ES) Internal API incompatibilities (to take advantage of rolling update capabilities, upgrade ROR KBN first)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**⚠️Warning** (ES) Support for ES < 6.8.0 was dropped


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) User settings available for all access type users


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Add option to change the Default Route and Time zone in User settings


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide correlation ID to Kibana logs


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) Rich, context-based debug logging in the LDAP connector and LDAP-related rules


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) Additional [validations](https://docs.readonlyrest.com/elasticsearch#configuring-an-acl-with-filter-fields-rules-when-using-kibana): `kibana` rule should not be used with some other rules in the same block


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Sometimes reports are not generated correctly for Kibana < 8.0.0 and the "Max attempt reached" error appears


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Adjust interactive API swagger dark mode colors


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) CSRF problem when multiple ECK Kibana instances


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Plugin doesn't run for a version Kibana < 7.11.0 when the OIDC proxy is enabled


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Session probe should log out the user when empty metadata was returned from ES ROR


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Misc issues when `xpack.security.enabled: true` is set


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Patched files permission issue

### (2024-03-15) What's new in **ROR 1.56.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Provide a way to switch light/dark mode per user


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.13.2, 8.13.1, 8.13.0, 7.17.20, 7.17.19 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.13.2, 8.13.1, 8.13.0, 7.17.20, 7.17.19 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**⚠️Warning** (ES) [for ES > 6.5 patching is required since this version of ROR](https://docs.readonlyrest.com/elasticsearch#id-5.-patch-elasticsearch)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) The activation key will be revalidated in the interval


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide a way to define Activation key [retrieval mode](https://docs.readonlyrest.com/v/develop/universal-builds#change-activation-key-retrieval-mode-via-kibana.yml)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Sometimes reports are not generated correctly for Kibana >= 8.0.0 and "Max attempt reached" error  appears


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) The OIDC scope configuration property was not applied and the default configuration was used instead.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) The OIDC proxy parameter was not handled properly in case of HTTPs connection over HTTP proxy server


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Missing information when Kibana is not patched


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Repositories and Snapshots handling by ES coordinating nodes](https://forum.readonlyrest.com/t/snapshot-status-cannot-modify-incoming-request/2471)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Internode SSL `certificate_verification: true` was causing problems with nodes discovery](https://forum.readonlyrest.com/t/upgrade-elasticsearch-8-2-to-8-x-leads-to-ssl-problems/2480)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Missing `x-elastic-product` header in the response when `fields` and `filter` rules were used


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Proper `forbid` policy handling during processing ROR login request


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) `application/nd-json` media type handling (in case of ES `7.x` versions)

### (2024-01-29) What's new in **ROR 1.55.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2023-51074](https://nvd.nist.gov/vuln/detail/CVE-2023-51074)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.12.2 ,8.12.1, 7.17.18, 7.17.17 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.12.2, 8.12.1, 7.17.18 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [Elasticsearch images with preinstalled ReadonlyREST plugin in Docker Hub](https://hub.docker.com/r/beshultd/elasticsearch-readonlyrest)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Optional `readonlyrest_kbn.auth.oidc_kc.proxyURL` kibana.yml configuration for the OIDC connection which allows declaring your proxy URL


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Upon successful activation and edition changes all sessions are cleared and users are logged out


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Saved objects are not visible for the users on Kibana >= 8.8.0


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [LDAP nested group IDs are properly escaped](https://forum.readonlyrest.com/t/support-kbn-ent-ldap-and-parentheses/2466)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Logout when a user with restricted `kibana.access` tried to see a restoration status of snapshots in Kibana

### (2023-12-17) What's new in **ROR 1.54.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [Scroll API: protected data could leak when the `fields` rule was used with `fls_engine` set to `es` or `es_with_lucene`](https://forum.readonlyrest.com/t/field-rule-not-working-when-exceeding-a-certain-no-of-docs/2415/7)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.12.0, 8.11.4 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.12.0, 8.11.4, 7.17.17 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide automatic [cleaning of stale sessions](https://docs.readonlyrest.com/kibana#automatic-session-cleanup)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide automatic cleaning of stale CSRF cookies


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Adjust the ROR API POST license endpoint body to the contract to respect the `license` body parameter instead of a `token`


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) `CorelationId`` is changed on every session refresh


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) ["missing authorization info" problem in some situations when `xpack.security.enabled` was configured to be `true`](https://forum.readonlyrest.com/t/diana-eck/2298/75)

### (2023-11-20) What's new in **ROR 1.53.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2023-4586](https://nvd.nist.gov/vuln/detail/CVE-2023-4586), [CVE-2023-5072](https://nvd.nist.gov/vuln/detail/CVE-2023-5072)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.11.3, 8.11.2, 8.11.1, 8.11.0, 7.17.16 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.11.3, 8.11.2, 8.11.1, 8.11.0, 7.17.16 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide Activate license endpoint to the ReadonlyREST API


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) [when the `kibana` rule and the `indices` rule are defined in the same block](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#index), there is no need to explicitly allow kibana-related indices


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) problem with reports generation when `kibana.index` in kibana.yml is used


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) crash loop during license service initialization


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) problem with logging in in KBN 7.17.13 (and above) and 8.10.4 (and above) when deployed using ECK


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [problem with multi-tenancy and ECK](https://forum.readonlyrest.com/t/multi-tanancy-issue/2427)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) problem with forbidden `/_create/config` response on Login to the Kibana


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [patching fix, when a non-default ES path is used (e.g. on K8s)](https://forum.readonlyrest.com/t/getting-java-lang-illegalargumentexception-when-initializing-ror-in-es-8-10-4/2441)

### (2023-10-09) What's new in **ROR 1.52.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2023-4586](https://access.redhat.com/security/cve/cve-2023-4586)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.10.4, 8.10.3, 7.17.15, 7.17.14 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.10.4, 8.10.3, 7.17.15, 7.17.14 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [New `token_authentication` rule](https://docs.readonlyrest.com/elasticsearch#token_authentication)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Permanently hide Kibana|ES features that are impossible to support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) [License expiration reminder](https://forum.readonlyrest.com/t/license-expiration-reminder/2417)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Make `kibana.index` setting from kibana.yml an invalid property for an Enterprise user


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Issue with not adding `elasticsearch.customHeaders` setting from kibana.yml to ROR requests


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Logout after opening Stack management Upgrading assistant


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Problem with logging in of two users in two tabs when two Kibana instances are used


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Problem with logging in when multi-tenancy is enabled and the `indices` rule is defined in the ROR settings

### (2023-09-25) What's new in **ROR 1.51.1**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [`fields` rule didn't work well in the case of ES 7.10.0 and later and more than 10 documents in the response](https://forum.readonlyrest.com/t/field-rule-not-working-when-exceeding-a-certain-no-of-docs/2415)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) issue with Observability Overview-based applications hiding


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Correct `kibana.index` handling for KBN >= 7.9.0 when multi-tenancy is disabled or unavailable


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Unrestricted Kibana Access on the tenancy switch when a selected tenant is not available anymore


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Unhandled error during login when `multiTenancyEnabled: false`


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) LDAP connectivity improvements

### (2023-09-10) What's new in **ROR 1.51.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (KBN) the issue with [api_only](https://docs.readonlyrest.com/elasticsearch#kibana-related-rules) access level user and accessing via Kibana UI


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.10.2, 8.10.1, 8.9.2, 7.17.13 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.10.2, 8.10.1, 8.10.0, 8.9.2, 7.17.13 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [Dynamic variables transformation support](https://docs.readonlyrest.com/elasticsearch#variables-functions)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Expose interactive Swagger as a new Security settings tab


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide detailed information about the invalid activation key


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) additional `hide_apps` validation in the `kibana` rule


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) the issue with the persistence of an activation key provided via UI when `readonlyrest_kbn.cookiePass` was not provided. The [readonlyrest_kbn.cookiePass](https://docs.readonlyrest.com/kibana#configuring-kibana) is required `kibana.yml` property


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) issues for Kibana versions between 7.9.0 and 7.10.2, related to the activation key, Spaces, and readonlyREST menu crash


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) The issue with a logout from Kibana when the link to the Kibana is open from a third-party application like `Gmail`


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [getting data streams when not full names of backing indices are declared in the `indices` rule](https://forum.readonlyrest.com/t/forbidden-for-creating-component-templates/2372/7)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) stack-management screen fix in case of `xpack.security.enabled: true`

### (2023-07-25) What's new in **ROR 1.50.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN/ES) ECK support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.9.1, 8.9.0, 7.17.12 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.9.1, 8.9.0, 7.17.12 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Introduce the new ReadonlyREST API


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Remove application item info from URL on the tenant switch to avoid a 404 not found message


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide Reordering available tenancies for proxy auth authentication


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide information about granted/rejected log-in users to debug logs

### (2023-06-27) What's new in **ROR 1.49.1**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2023-2976](https://nvd.nist.gov/vuln/detail/CVE-2023-2976)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2023-34462](https://github.com/advisories/GHSA-6mjq-h674-j845)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.8.2, 8.8.1, 8.8.0, 7.17.11 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.8.2, 7.17.11 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [LDAP nested groups support](https://docs.readonlyrest.com/elasticsearch#ldap-connector)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) [Allow setting default tenancy via `/login?defaultGroup` query param. To be used with "Custom Middleware" feature for reordering available tenancies in the ROR menu](https://docs.readonlyrest.com/examples/custom-middleware/reordering-available-tenancies)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Fix for ES warnings in logs about custom action names (ROR internal actions)](https://forum.readonlyrest.com/t/invalid-action-name-cluster-ror-audit-event-put/2186)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [kibana access `rw` and `admin` should allow to manage component templates](https://forum.readonlyrest.com/t/forbidden-for-creating-component-templates/2372)

### (2023-05-28) What's new in **ROR 1.49.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.8.1 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Handle `elasticsearch.serviceAccountSupport` configuration property


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide a way to Hidden apps Stack management items hiding


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide an automated migration of tenancy indices on major Kibana version upgrade


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) external group ID patterns support in the external to local groups mapping


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) the issue with the replica number being set to 0 on tenant index creation


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) users won't log out from Kibana on the 500 status error


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) the issue with Kibana keystore not being read by the Kibana plugin


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN < 7.9.0) logging issue when two Kibanas are handled by one browser at the same time


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) resolving ENVs to YAML number in ROR settings

### (2023-04-15) What's new in **ROR 1.48.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2022-45688](https://nvd.nist.gov/vuln/detail/CVE-2022-45688)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.7.1, 7.17.10 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.8.0, 8.7.1, 7.17.10 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN/ES) [Introducing "Custom Middleware" functionality](https://docs.readonlyrest.com/kibana#custom-middleware)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN/ES) [`allowed_api_paths` support in the `kibana` ACL rule](https://docs.readonlyrest.com/elasticsearch#kibana-related-rules)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Add CSRF protection in the login form


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Restore deprecated "kibana.index" support for Kibana > 8.x


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [all Kibana-related rules are gathered in one, new `kibana` ACL rule](https://docs.readonlyrest.com/elasticsearch#kibana-related-rules)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [audit supports a new output type: `log`](https://docs.readonlyrest.com/elasticsearch/audit)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Provide a way to disable multi-tenancy in ROR Enterprise


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Realign index templates behaviour to the old platform


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Error logs when SAML obtains an unusable username from the assertion


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Test configuration warnings improvement


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) [Added support to override default response code for not started ROR](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/issues/794)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Security card not hidden by default


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Hidden apps regex with two "or" operators don't hide all kibana apps


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fix Alerting Rules resulting in logout issue


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fix audit dashboard


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Stop handling 500 error from `api/lens/existing_fields`


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fix lens app


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN < 7.9.x) using a custom kibana index in cooperation with ROR Free

### (2023-02-13) What's new in **ROR 1.47.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) "/" endpoint was not protected for ES 8.x


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) "/_cat" endpoint was not protected for all ES versions


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.7.0, 8.6.2 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.7.0, 8.6.2 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [the `data_streams` rule](https://docs.readonlyrest.com/v/develop/elasticsearch#data_streams)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) optimisation in hidden apps feature


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Opening index management mappings tab forces logout


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Fix dark mode in the ROR menu


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) YAML editor updates and fixes


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Data streams support in the `indices` rule


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) NPE when `_search` with aggregations (script) and the `fields` rule were used together

### (2023-01-02) What's new in **ROR 1.46.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2022-1471](https://nvd.nist.gov/vuln/detail/CVE-2022-1471), [CVE-2022-41915](https://nvd.nist.gov/vuln/detail/CVE-2022-41915), [CVE-2022-36944](https://nvd.nist.gov/vuln/detail/CVE-2022-36944) in [audit Scala 2.13 jar](https://mvnrepository.com/artifact/tech.beshu.ror/audit)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.6.1, 8.6.0, 7.17.9 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.6.1, 8.6.0, 7.17.9 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Activation key management UI


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Less verbose logging in info mode


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) "Stack management" kibana compatibility


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Test settings pop up won't show


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) hide apps behaviour when "Management" is hidden


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Data view with a ":" symbol forces logout from a kibana


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Session probe causes constant refresh when no `kibana_access` defined


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) large report generation using data from a remote cluster with enabled x-pack security

### (2022-12-05) What's new in **ROR 1.45.1**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.5.3, 7.17.8 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.5.3, 7.17.8 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) ROR KBN patching script

### (2022-11-29) What's new in **ROR 1.45.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2022-42003](https://nvd.nist.gov/vuln/detail/CVE-2022-42003), [CVE-2022-45146](https://nvd.nist.gov/vuln/detail/CVE-2022-45146)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Activation Key API: read AK from ROR_ACTIVATION_KEY.txt


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Activation Key API: submit AK via POST /pkp/license (Basic auth)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Inject CSS/JS files in login page


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Add user metadata to <body> for extra UI customization


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) Added groups_and mode to [groups_provider_authorization](https://docs.readonlyrest.com/elasticsearch#groups_provider_authorization) rule


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) all authorization rules support wildcards in group IDs


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) connections in the LDAP pool should not be closed unnecessarily


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Deterministic reporting index detection


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Move free type impersonation to the local users area


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) don't logout when initial JWT token expires


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Direct Kibana API requests not aware of kibana_index


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) RO and RO_strict kibana accesses


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [when `fls_engine: es` is configured and `fields` rule is used, aggregations should be available only for allowed fields](https://forum.readonlyrest.com/t/field-level-security-and-aggregations/2133)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Data streams creation issue fix](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/issues/829)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) Unknown structure of index settings issue fix


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) resolving index names with wildcards should take into consideration the current index state and request indices options

### (2022-10-09) What's new in **ROR 1.44.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2022-25857](https://nvd.nist.gov/vuln/detail/CVE-2022-25857)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.5.2, 8.5.1, 8.5.0, 7.17.7 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.5.2, 8.5.1, 8.5.0, 7.17.7 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) **plugin packages are now [universal](https://docs.readonlyrest.com/universal-builds)**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) **Manage your activation keys through the [customer portal](https://readonlyrest.com/customer)**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) Added support for certificates in PEM format


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) SAML groups list duplication made header size exceed limits


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) kibana_access: admin has now privileges to manage a Kibana cluster


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) added distributed and persistent Test Settings & Auth Mocks configuration for the Impersonation Feature


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) handling high load when LDAP rules are used


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) `client_authentication` settings in internode SSL configuration


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) `acl:available_groups` dynamic variable can be used in a single value context


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) SNI handling (internode SSL)

### (2022-08-22) What's new in **ROR 1.43.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) 8.4.3, 8.4.2, 8.4.1, 8.4.0, 7.17.6 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) 8.4.3, 8.4.2, 8.4.1, 8.4.0, 7.17.6 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) `kibana_custom_js_inject_file` feature


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [`ror-tools` fix for Windows OS (patching ES 3.x issue)](https://forum.readonlyrest.com/t/ror-plugin-for-es-8-x-patch-error/2115)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) resolving indices in the remote x-pack cluster


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN|PRO) ROR menu title wraps when version text is too short (cosmetic)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) infinite loading when kibana_access not defined for user


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) transient error with randomly choosing off range bind port on localhost


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) 404 on login when `xpack.spaces.enabled: false`

### (2022-07-25) What's new in **ROR 1.42.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN|ES) 8.3.3, 8.3.2, 8.3.1, 8.3.0, 7.15.5 support


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Search box in tenancy switcher (when #tenancies > 5)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) added configuration warnings in the Impersonation Feature


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Logout didn't delete the SAML session on the IdP


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) 5xx errors from Elasticsearch break Kibana users' session unrecoverably


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) ROR node cooperation with X-pack nodes

### (2022-06-21) What's new in **ROR 1.41.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) Added `groups_and` mode to [`ror_kbn_auth`](https://docs.readonlyrest.com/elasticsearch#ror_kbn_auth) and [`jwt_auth`](https://docs.readonlyrest.com/elasticsearch#jwt_auth) rules


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Prevent native credentials dialogue to appear in Kibana when ES responds 401


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Logging in after logout shows the same page you last visited


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) x-ror-correlation-id header lets you audit a whole Kibana session


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES|KBN) tenancy selector didn't work well with `jwt_auth` and `ror_kbn_auth` rules


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Support for special characters in tenancy names


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) OIDC logout flow redirecting to bad request error


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) OIDC connector not working in Kibana < 7.12.0

### (2022-05-24) What's new in **ROR 1.40.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2022-25647](https://nvd.nist.gov/vuln/detail/CVE-2022-25647) & [CVE-2022-24823](https://nvd.nist.gov/vuln/detail/CVE-2022-24823) & [CVE-2020-13956](https://nvd.nist.gov/vuln/detail/CVE-2020-13956) & [CVE-2020-36518](https://nvd.nist.gov/vuln/detail/CVE-2020-36518) &  [CVE-2020-13956](https://nvd.nist.gov/vuln/detail/CVE-2020-13956) & [CVE-2020-36518](https://nvd.nist.gov/vuln/detail/CVE-2020-36518)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (KBN) "Security" app not entirely hidden in 8.2.x


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) New Support for 8.2.3, 8.2.2, 8.2.1, 7.17.4


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) New Support for 8.2.2 8.2.1, 7.17.4


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES & KBN) [The Impersonation feature](https://docs.readonlyrest.com/kibana#impersonation)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [FIPS compliant SSL mode](https://docs.readonlyrest.com/elasticsearch/fips)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) SAML cert is now required


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) moved OIDC to better library


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) OIDC jwksURL is now required


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) `indices: ["1"]` interpreted as integer and fails to parse


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) /login?jwt=xxx authorization now works again


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) OIDC/SAML assertion claims were not forwarded to ES


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) include whitelisted headers while logging


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) basepath handling fixes (too many redirects)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Make ROR default space the actual default one


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) OIDC connection error

### (2022-03-19) What's new in **ROR 1.39.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (KBN) XSS sanitize path requested


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2020-36518](https://nvd.nist.gov/vuln/detail/CVE-2020-36518) & [CVE-2022-21653](https://nvd.nist.gov/vuln/detail/CVE-2022-21653)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) New Support for 8.2.0 8.1.3, 8.1.2, 8.1.1, 8.1.0, 8.0.0, 8.0.1, 7.17.3, 7.17.2


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) New Support for 8.2.0, 8.1.3, 8.1.2, 8.1.1, 8.1.0, 8.0.0, 8.0.1 ([required additional patching step](https://docs.readonlyrest.com/elasticsearch#3.-patch-es))


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) New Support for 7.17.3, 7.17.2


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [New `groups_and` ACL rule](https://docs.readonlyrest.com/elasticsearch#groups_and)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Stop inlining whitelisted headers into Authorization header


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Log additional errors and info related to HA


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Misc internal dependencies upgrades


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Mandatory elasticsearch credentials in kibana.yml


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [Reporting page redirect on refresh when kibana_hide_apps: ["Stack Management"]](https://forum.readonlyrest.com/t/when-hiding-stack-management-a-redirect-appears-with-report/2088)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) whitelistedPaths: log errors when 404 occurs


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [Issue uploading large payload](https://forum.readonlyrest.com/t/issue-uploading-large-payload/2091)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) `elasticsearch.requestHeadersWhitelist` should be case insensitive


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Issue with handling data streams by `indices` rule](https://forum.readonlyrest.com/t/ror-1-37-0-indices-rule-and-alias-within-kibana/2078)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) X-Pack SSL nodes cooperation with ROR SSL nodes


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) _msearch issue when filter rules was used in matched block

### (2022-01-17) What's new in **ROR 1.38.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) New Support for 7.17.0, 7.17.1


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) New Support for 7.17.0


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [Configuration for custom audit cluster](https://github.com/beshu-tech/readonlyrest-docs/blob/v1.38.x/elasticsearch.md#custom-audit-cluster)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) Separate "audit" section for all audit settings


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Editor rendering issue with kibana basePath enabled

### (2021-12-14) What's new in **ROR 1.37.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2021-43797](https://nvd.nist.gov/vuln/detail/CVE-2021-43797)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) New Support for 7.16.3, 7.16.2, 6.8.23, 6.8.22


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) New Support for 7.16.3, 7.16.2, 7.16.1, 7.16.10, 6.8.23, 6.8.22, 6.8.21


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (ES) fields rule handling in the context of x-Pack SQL requests


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) filter rule handling in the context of x-Pack SQL requests


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) POST / bulk cause an 400 error in devtools console


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) More robust Kibana patcher + better logs messages

### (2021-11-21) What's new in **ROR 1.36.0**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) New Support for 7.16.1, 7.16.0, 6.8.21


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Support Kibana 7.15.2


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) [Added support for setting up cluster containing ES with ROR (with disabled XPack security) and ES with XPack security enabled](https://forum.readonlyrest.com/t/ssl-internode-with-elk-cluster/1916)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) kibana_hide_apps: [ror|kibana] to remove kibana mgmt button


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [/_snapshot/_status should return only running snapshots](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/issues/756)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (ES) [Adding policy to index template bug](https://forum.readonlyrest.com/t/forbidden-by-readonlyrest-es-plugin-with-add-policy-to-index-template-action-in-kibana/1969)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Index management tabs result in "forbidden" error


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [corrupted patch file for Kibana 7.9.x](https://forum.readonlyrest.com/t/ror-1-35-1-kibana-7-9-3-unable-to-patch/2018)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [YAML editor not working in air-gapped environments](https://forum.readonlyrest.com/t/readonlyrest-security-settings-editor-loading/2014/5)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [Devtools not working](https://forum.readonlyrest.com/t/kibana-devtools-error-does-not-support-having-a-body/2027)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) [Monitoring not working in multi-tenancy](https://forum.readonlyrest.com/t/kibana-alerting-not-working-with-readonlyrest/1986)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Regression in Kibana < 6.8.x front end crash


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Kibana < 7.8.x prevent navigation to hidden apps from home links


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Kibana < 7.8.x implicitly hide kibana:dashboard when kibana:dashboards is hidden (and viceversa)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Kibana < 7.8.x broken `clearSessionOnEvents: [tenancyHop]`

### (2021-10-17) What's new in **ROR 1.35.1**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚨Security Fix** (ES) [CVE-2021-21409](https://nvd.nist.gov/vuln/detail/CVE-2021-21409) & [CVE-2021-27568](https://nvd.nist.gov/vuln/detail/CVE-2021-27568)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (KBN) Support Kibana 7.15.1


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🚀New** (ES) New Support for 7.15.2


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Support "server.ssl.supportedProtocols" settings


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Support "server.ssl.cipherSuites"


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🧐Enhancement** (KBN) Always honor SSL cipher order


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) Don'thide "Add/Remove field as column" in Discover app for RO users


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**🐞Fix** (KBN) More alerting fixes (only for main tenancy)

### (2021-10-12) What's new in **ROR 1.35.0**


### (2021-09-24) What's new in **ROR 1.34.0**


### (2021-08-14) What's new in **ROR 1.33.1**


### (2021-08-09) What's new in **ROR 1.33.0**


### (2021-07-25) What's new in **ROR 1.32.0**

 
### (2021-06-29) What's new in **ROR 1.31.0**


### (2021-05-26) What's new in **ROR 1.30.1**


### (2021-05-16) What's new in **ROR 1.30.0**


### (2021-04-09) What's new in **ROR 1.29.0**


### (2021-04-01) What's new in **ROR 1.28.2**


### (2021-03-24) What's new in **ROR 1.28.1**


### (2021-03-14) What's new in **ROR 1.28.0**


### (2021-02-27) What's new in **ROR 1.27.1**


### (2021-02-16) What's new in **ROR 1.27.0**


### (2021-01-11) What's new in **ROR 1.26.1**


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
