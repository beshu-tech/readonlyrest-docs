# Elastic Fleet with ReadonlyREST

[Elastic Fleet](https://www.elastic.co/guide/en/fleet/current/fleet-overview.html) manages Elastic Agents centrally through Kibana. When Fleet is set up, it creates two kinds of dynamic Elasticsearch credentials that ReadonlyREST needs to recognize and validate:

- **Service tokens** - used by Fleet Server to authenticate with Elasticsearch. These are created by Kibana during Fleet setup and belong to Elasticsearch's built-in `elastic/fleet-server` service account.
- **API keys** - issued to each enrolled Elastic Agent. Fleet Server creates and rotates these automatically; each agent uses its own key to ship data.

Because both credential types are generated at runtime (not known in advance), they cannot be matched with static `auth_key` or `auth_key_sha256` rules. Instead, ReadonlyREST's `token_authentication` rule with `type: service-token` or `type: api-key` delegates validation to Elasticsearch, which has the ground truth for both.

## ReadonlyREST settings

```yaml
readonlyrest:
  access_control_rules:

    # 1. Kibana user - used by Kibana itself and by Fleet initialisation scripts.
    #    No action or index restriction; Kibana needs unrestricted access during
    #    Fleet setup (e.g. bootstrapping the Fleet Server service account).
    - name: "KIBANA"
      type: allow
      auth_key: kibana:kibana

    # 2. Fleet Server - authenticates using an Elasticsearch service token.
    #    ReadonlyREST validates the token against Elasticsearch's service account API.
    #    No action restriction: Fleet Server needs to call
    #    cluster:admin/xpack/security/api_key/create to issue API keys to
    #    enrolling agents.
    - name: "Fleet server"
      type: allow
      token_authentication:
        type: "service-token"
        username: "fleet"
      indices:
        - ".fleet-servers"
        - ".fleet-agents"
        - ".fleet-actions"
        - ".fleet-policies"
        - ".fleet-policies-leader"
        - ".fleet-enrollment-api-keys"

    # 3. Elastic Agents - each agent authenticates with its own API key, issued
    #    and rotated by Fleet Server. ReadonlyREST validates the key against Elasticsearch
    #    and grants access to the observability data-stream indices.
    - name: "Agents"
      type: allow
      token_authentication:
        type: "api-key"
        username: "fleet"
      indices:
        - ".apm-agent-configuration"
        - "metrics-*"
        - "traces-*"
        - "logs-*"

    # 4. Forbid direct token management - only Kibana and Fleet Server (matched
    #    above) should create or revoke service tokens and API keys. This block
    #    denies these actions for everyone else.
    - name: "Forbid access to service accounts and API keys"
      type: forbid
      actions:
        - "cluster:admin/xpack/security/service_account/*"
        - "cluster:admin/xpack/security/api_key/*"

    # 5. Admin user - full Kibana access.
    - name: "Admins"
      type: allow
      auth_key: admin:admin
      kibana:
        access: admin
```

## How Fleet credentials flow through ReadonlyREST

1. **Kibana creates a service token** - during Fleet setup, Kibana calls `cluster:admin/xpack/security/service_account/*` to create the Fleet Server service token. This request is authenticated by the `KIBANA` block.
2. **Fleet Server creates API keys** - Fleet Server uses its service token to call `cluster:admin/xpack/security/api_key/create`, issuing an API key to each enrolling agent. This request is authenticated by the `Fleet server` block.
3. **Elastic Agents use their API keys** - each agent presents its API key on every request to ship data to Elasticsearch. These requests are authenticated by the `Agents` block.

## Why the `forbid` block is necessary

Only Kibana and Fleet Server should be able to create service tokens and API keys - no other user needs these actions. The `KIBANA` and `Fleet server` blocks already permit these calls for the accounts that legitimately need them. The `forbid` block sits below those blocks and denies any remaining request that targets service-account or API-key management actions, preventing other authenticated users from creating, revoking or listing credentials.

## Credential rotation

You do not need to put service tokens or API key values into `readonlyrest.yml`. ReadonlyREST never sees or stores them - it asks Elasticsearch to validate each token on the fly. This means:

- Fleet Server can rotate its service token without any ReadonlyREST config change.
- Agents can be enrolled, unenrolled, and re-keyed without touching ReadonlyREST.
- The only things that must stay in sync with your deployment are the **index patterns** in the `service-token` and `api-key` blocks.

## Running the example

A full working example with Elasticsearch, Kibana (both with ReadonlyREST), Fleet Server, an Elastic Agent (APM), a demo Node.js app, and a traffic simulator is available in the [readonlyrest-examples](https://github.com/beshu-tech/readonlyrest-examples/tree/master/examples/fleet) repository:

```bash
curl -sL https://raw.githubusercontent.com/beshu-tech/readonlyrest-examples/master/quickstart.sh | bash -s fleet
```

Once running, log into Kibana and navigate to **Management → Fleet** to see the enrolled agent and its policy, or to **Observability → APM** for traces from the demo application.
