# Universal Builds

Starting from ReadonlyREST 1.44.0, our Kibana plugins are released in a unified format.

## What is a universal build?

A universal build is a single Kibana plugin deliverable that is compatible with a given Kibana version.

![](assets/universal\_build.png)

If not activated, a universal build will behave exactly like the old "Free" edition.

Some activation keys can be used to unlock PRO and Enterprise features. If a trial activation key is used, these features availability will be limited in time.

## How to install a universal build?

The same way it worked before, just download it the same way you used to, from the [Downloads page](https://readonlyrest.com/download) and install it as usual.

## How to activate PRO/Enterprise features a universal build?

Check out our new [customer portal](https://readonlyrest.com/customer). If your email is associated to a customer contract with a valid ongoing subscription, you will be able to obtain an activation key. Otherwise, you will be able to obtain a trial activation key.

There are a few ways to pass the activation key to Kibana. Regardless of which one you use, the result will be that Kibana will save the activation key to an encrypted Elasticsearch index, so other Kibana instances will pick up the new activation key.

### Via Environmental variable

This method is useful for Docker deployments. Just set the `ROR_ACTIVATION_KEY` environment variable to the activation key you obtained from the customer portal.

```bash
$ export ROR_ACTIVATION_KEY=<your activation key>
$ bin/kibana 
```

### Via our plugin license API

Once Kibana is up and running, you can send a HTTP request to Kibana.

```
POST http://<kibana-host-with-ror>:5601/api/ror/license?overwrite=true
{
  "token": "your activation key" 
}
```

### Via ROR\_ACTIVATION\_KEY.txt file

The universal build is a zip archive. You can add a small text file with your activation key to this archive before installing it, so it will be picked up automatically at the first boot.
- The text file should be called `ROR_ACTIVATION_KEY.txt`
- It should contain your secret activation key string (no spaces, no new lines)
- It should be added to the **root directory** of the plugin zip archive

Now install the plugin file, start Kibana, and the activation key should be loaded.


### Interactive activation key management

This method is useful for manual deployments.

1. Start Kibana in the default "Free" edition mode
2. Login in Kibana as an `admin` or `unrestricted` kibana access
3. Toggle the ROR Menu on the top right
4. Click on the "Free" text tag

![](assets/edit\_activation\_key.png)

1. Enter the activation key you obtained from the customer portal in the license management UI

![](assets/activation\_keys\_gui.png)
