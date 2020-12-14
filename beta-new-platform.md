---
description: ReadonlyREST Enterprise
---

# \[Beta\] Enterprise New Platform

## What's different?

Kibana is a fast changing piece of software. In the past, it was very frequent that a new Kibana release would break or introduce regressions in our product, and fixing and adapting has been an endless manual work for our team. 

Today, the new ReadonlyREST Enterprise has been completely re-enginered to be more robust to changes, modular and testable.

### Will users notice different UX?

Only subtle differences, and a few of these are total improvements.

For example, the ROR Admin Kibana app has now become a modal window. Opens and closes quickly, and the Kibana navigation does not change.  

### Will our deployment scripts need a tweak?

The answer is: **yes they will, but only one extra command.**

This is because the new ROR owes its extra powers to a tiny bit of code injection in a couple of Kibana core files. And for legal reasons you have to patch them yourself using our patcher/unpatcher "ror-tools.js". 

#### Installation

```bash
# Normal installation
$ bin/kibana-plugin install file:///tmp/readonlyrest_kbn_v1.27.0_es7.10.1.zip
...
# [NEW!] Patch Kibana core files 
$ node/bin/node plugins/readonlyrest_kbn/ror-tools.js patch

```

{% hint style="info" %}
Don't forget to launch "unpatch" before updating the plugin or Kibana.  
Even better, throw away the entire Kibana directory, and install from scratch. Like Docker would.
{% endhint %}

#### Uninstall

```bash
# Uninstall normally
$ bin/kibana-plugin remove readonlyrest_kbn
...
# [NEW!] Un-patch Kibana core files 
$ node/bin/node plugins/readonlyrest_kbn/ror-tools.js unpatch

```



