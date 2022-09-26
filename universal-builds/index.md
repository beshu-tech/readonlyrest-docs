# Universal builds (kibana plugin)

Starting from ReadonlyREST 1.44.0, our Kibana plugins are released in a unified format:

| No more                                                                                                                                   | Now                                           |
|-------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------|
| readonlyrest_kbn_free-1.43.0_es8.4.1.zip<br>readonlyrest_kbn_pro-1.43.0_es8.4.1.zip<br>readonlyrest_kbn_enterprise-1.43.0_es8.4.1.zip<br>readonlyrest_kbn_pro-1.43.0-20220902_es8.4.1.zip | readonlyrest_kbn_universal-1.44.0_es8.4.1.zip |

## What is a universal build?

A universal build is a single Kibana plugin deliverable that is compatible with a given Kibana version.
If not activated, a universal build will behave exactly like the old "Free" edition.

Some activation keys can be used to unlock PRO and Enterprise features. If a trial activation key is used, these features availability will be limited in time. 

## How to install a universal build?
The same way it worked before, just download it the same way you used to, from the [Downloads page](https://readonlyrest.com/download) and install it as usual.


## How to activate PRO/Enterprise features a universal build?
Check out our new [customer portal](https://readonlyrest.com/customer). If your email is associated to a customer contract with a valid ongoing subscription, you will be able to obtain an activation key.
Otherwise, you will be able to obtain a trial activation key.

There are two ways to pass the activation key to Kibana:

### Via Environmental variable
This method is useful for Docker deployments. Just set the `ROR_ACTIVATION_KEY` environment variable to the activation key you obtained from the customer portal.
```bash 
$ export ROR_ACTIVATION_KEY=<your activation key>
$ bin/kibana 
```

### Interactive activation key management
This method is useful for manual deployments.

1. Start Kibana in the default "Free" edition mode
2. Login in Kibana as an `admin` or `unrestricted` kibana access
3. Toggle the ROR Menu on the top right
[](assets/edit_activation_key.png)
4. Click on the "Free" text tag
[](assets/activation_keys_gui.png)
5. Enter the activation key you obtained from the customer portal in the license management UI
