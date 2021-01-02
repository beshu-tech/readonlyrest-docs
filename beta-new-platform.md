# \[Beta\] Enterprise New Platform

## How can I get a build?

This is a closed beta for the current Enterprise subscribers. If you are one of them, please:

* **email us at**: support AT readonlyrest DOT com 
* **with subject**: "ROR Enterprise BETA builds for &lt;Your Company Name&gt;"

We will be grateful for your kind feedback.

## How can I get updates?

Simple: head to [beta.readonlyrest.com](https://beta.readonlyrest.com) and subscribe to the announcements. 

## What's different?

Kibana is a fast changing piece of software. In the past, it was very frequent that a new Kibana release would break or introduce regressions in our product, and fixing and adapting has been an endless manual work for our team. 

Today, the new ReadonlyREST Enterprise has been completely re-enginered to be more robust to changes, modular and testable.

### Will users notice different UX?

Only subtle differences, and many of these are just total improvements.

For example, our administrative Kibana app has now become a modal window. It opens and closes quickly, and does not navigate your session away from what you were doing in Kibana.

### Will our deployment scripts need a tweak?

The answer is: **yes they will, but only one extra command.** See installation/uninstallation instructions below.

## **Install and uninstall**

This is because the new ROR owes its extra powers to a tiny bit of code injection in a couple of Kibana core files. And for legal reasons you have to patch them yourself using our patcher/unpatcher "ror-tools.js". 

#### Installation

With the new plugin, you need a post-installation step. This will slightly modify some core Kibana files.

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

There's also a pre-uninstallation step, to bring back the Kibana files to the original state.

```bash
# Uninstall normally
$ bin/kibana-plugin remove readonlyrest_kbn
...
# [NEW!] Un-patch Kibana core files 
$ node/bin/node plugins/readonlyrest_kbn/ror-tools.js unpatch

```



