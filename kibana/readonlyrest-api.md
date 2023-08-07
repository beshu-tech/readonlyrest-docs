---
description: >-
  An authenticated API for changing the security settings without rebooting the
  ES cluster.
---

# ReadonlyREST API

As an Enterprise user, you can benefit from automating the security configuration changes without the need to reboot the ES cluster.

Every request is **validated against ROR syntax first**, and rejected if syntactically or semantically incorrect. This adds to the safety of each change operation.

{% swagger src="https://raw.githubusercontent.com/beshu-tech/readonlyrest-docs/develop/.gitbook/assets/ReadonlyREST-api.yml" path="/api/ror/settings" method="post" %}
[https://raw.githubusercontent.com/beshu-tech/readonlyrest-docs/develop/.gitbook/assets/ReadonlyREST-api.yml](https://raw.githubusercontent.com/beshu-tech/readonlyrest-docs/develop/.gitbook/assets/ReadonlyREST-api.yml)
{% endswagger %}

{% swagger src="https://raw.githubusercontent.com/beshu-tech/readonlyrest-docs/develop/.gitbook/assets/ReadonlyREST-api.yml" path="/api/ror/settings/upload" method="post" %}
[https://raw.githubusercontent.com/beshu-tech/readonlyrest-docs/develop/.gitbook/assets/ReadonlyREST-api.yml](https://raw.githubusercontent.com/beshu-tech/readonlyrest-docs/develop/.gitbook/assets/ReadonlyREST-api.yml)
{% endswagger %}
