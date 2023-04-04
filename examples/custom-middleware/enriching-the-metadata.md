---
description: Enriching the metadata
---

# Enriching the metadata

The metadata is the user-specific data available after the Kibana user successfully logs in. Thanks to the custom middleware, you can enrich metadata and use them in the kibana custom js file.
For example to load a custom logo to the kibana you can:
1. Declare `readonlyrest_kbn.custom_middleware_inject_file: 'path/to/custom_middleware_inject_file.js'` in the kibana.yml and declare `custom_middleware_inject_file.js`

```ts
async function customMiddleware(req, res, next) {
  const metadata =
    req.rorRequest && req.rorRequest.getIdentitySession() && req.rorRequest.getIdentitySession().metadata;

  if (metadata && metadata.username === 'admin') {
    req.rorRequest.enrichIdentitySessionMetadata({
      newLogo:
        'PHN2ZyBpZD0ic3ZnIiB2ZXJzaW9uPSIxLjEiIHdpZHRoPSI0MDAiIGhlaWdodD0iMzYzIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiPgogICAgPGcgaWQ9InN2Z2ciPgogICAgICAgIDxwYXRoIGlkPSJwYXRoMCIKICAgICAgICAgICAgICBkPSJNMTIyLjgzNiAxMS42MTEgQyAxMTMuNTg2IDE0LjQwNCwxMDguMDAyIDkzLjQxNiwxMDkuOTgyIDE5My41MDAgQyAxMTEuNTE1IDI3MC45ODAsMTExLjI4OCAzMDAuNzQzLDEwOS4wODUgMzExLjI4MyBDIDEwNS4yNjkgMzI5LjU0NCwxMDAuNDI2IDMyNy4zNDAsOTYuMzczIDMwNS41MDAgQyA5NC44ODcgMjk3LjQ4OSw5NC42MzEgMjgxLjI3Nyw5NC4wNDMgMTU4LjAwMCBDIDkzLjM1NCAxMy40NTcsOTMuNDQzIDE2LjAwMCw4OS4wNjggMTYuMDAwIEMgNjcuMDkxIDE2LjAwMCwzMC42ODMgNDQuNjgwLDE5Ljc2MCA3MC41OTUgQyAxMS43NDggODkuNjA3LDkuMjk2IDEzMi42NTcsMTQuNzI1IDE1OS4wMDAgQyAzMC4xNjkgMjMzLjkzOCw1NC45MjIgMjg4LjYxNiw4Ny42MzYgMzIwLjA1OCBDIDEyMi4xNjAgMzUzLjIzOCwxNzAuOTYxIDM1Ny45MjAsMjMwLjAwMCAzMzMuNzE1IEMgMjQ3LjY5OSAzMjYuNDU5LDI0OC4yNjEgMzI1LjA5OCwyNDIuMjg0IDMwNC4wMDAgQyAyMjkuNzc2IDI1OS44NDYsMjE3LjE2OCAyMzkuMDE4LDE3Ni42MDQgMTk1LjUwMCBDIDE1My43NDYgMTcwLjk3OCwxNDkuMzkxIDE2NC4zMzQsMTQ1LjA4NiAxNDcuNDE5IEMgMTM3LjE3NyAxMTYuMzQ3LDE0MS4zMjcgOTIuMzg0LDE2My41MTcgNDEuMDAwIEMgMTc0LjkwNiAxNC42MjYsMTc0Ljg4OCAxNC40NjksMTYwLjM2OCAxMy4wMTUgQyAxNDguODIxIDExLjg2MCwxMjQuOTY4IDEwLjk2NywxMjIuODM2IDExLjYxMSBNMTk2LjA2MSAyMi4xNDEgQyAxOTUuNTE0IDIzLjQzOCwxOTMuMDU1IDI5LjkwMCwxOTAuNTk3IDM2LjUwMCBDIDE4OC4xNDAgNDMuMTAwLDE4My4yMTAgNTUuNTg2LDE3OS42NDQgNjQuMjQ3IEMgMTUzLjU3MiAxMjcuNTU5LDE1NC4wMjUgMTMxLjI3NCwxOTMuMDM2IDE3NC4wMDAgQyAyMjYuMjg4IDIxMC40MTksMjQ5Ljk2OSAyNTIuOTM2LDI1OC40ODEgMjkxLjUwMCBDIDI2MC44NTIgMzAyLjI0MywyNjIuNzkyIDMwOS4yMDksMjYzLjg3OCAzMTAuODc2IEMgMjY0Ljg4NiAzMTIuNDI1LDI3MC4wMzMgMzA5LjI5NCwyNzguNzI0IDMwMS44NDcgQyAyODEuMzUxIDI5OS41OTYsMjg2LjQyNSAyOTUuMjQ3LDI5MC4wMDAgMjkyLjE4MiBDIDMzMy40NTYgMjU0LjkzMCwzNzQuMTM0IDIwMS45NzMsMzgxLjkzMSAxNzIuNTAwIEMgMzkzLjczMiAxMjcuODkwLDMzNi4yMDMgNjguNTg2LDI0OS4wMDAgMzUuNDY3IEMgMjQ3LjA3NSAzNC43MzYsMjQzLjAyNSAzMy4xODMsMjQwLjAwMCAzMi4wMTUgQyAyMTMuNDEwIDIxLjc0OCwxOTcuNzE1IDE4LjIyMSwxOTYuMDYxIDIyLjE0MSAiCiAgICAgICAgICAgICAgc3Ryb2tlPSJub25lIiBmaWxsPSIjMDBiZmIyIiBmaWxsLXJ1bGU9ImV2ZW5vZGQiPjwvcGF0aD4KICAgIDwvZz4KPC9zdmc+Cg=='
    });
  }

  return next();
}
```

In this example, thanks to the enrichIdentitySessionMetadata method we can pass new logo custom metadata when the logged-in user username is 'admin'.
It mustn't be a static value, you can ask for external service for the logo:

```ts
if (metadata && metadata.username === 'admin') {
    const response = fetch(<EXTERNAL_SERVICE_URL>);
    req.rorRequest.enrichIdentitySessionMetadata({
      newLogo: await response.json().newLogo
    });
  }
```

Now, enriched metadata will be available in the custom kibana js script, where you can perform client based operations like logo replacement.

**⚠️IMPORTANT** Custom middleware must return `next()` function, to not block the request

2. To replace the logo, we need to declare the custom kibana js file `readonlyrest_kbn.kibana_custom_js_inject_file: '/path/to/custom_kibana.js'`

```js
const logoHeader = document.querySelector('.euiHeaderLogo');

if (window.ROR_METADATA.newLogo) {
  Array.from(logoHeader.childNodes).forEach(node => {
    node.style.display = 'none';
  });

  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      mutation.addedNodes.forEach(node => {
        const customLogo = document.querySelector('#customLogo');

        const createCustomLogo = () => {
          const img = document.createElement('img');
          img.src = `data:image/svg+xml;base64,${window.ROR_METADATA.newLogo}`;
          img.style.width = '32px';
          img.style.height = '32px';
          img.id = 'customLogo';
          logoHeader.appendChild(img);
        };

        const hideAllLogoElements = () => {
          Array.from(logoHeader.childNodes).forEach(node => {
            node.style.display = 'none';
          });
        };

        const handleInit = () => {
          hideAllLogoElements();
          createCustomLogo();
        };

        if (customLogo) {
          const displayCustomLogo = () => {
            customLogo.style.display = 'block';
          };
          const hideCustomLogo = () => {
            customLogo.style.display = 'none';
          };
          if (node.role === 'progressbar') {
            hideCustomLogo();
          }

          if (node.role === 'img') {
            const hideDefaultLogo = () => {
              node.style.display = 'none';
            };

            hideDefaultLogo();
            displayCustomLogo();
          }
        }

        if (node.dataset.type === 'logoElastic' && !customLogo) {
          handleInit();
        }
      });
    });
  });

  observer.observe(logoHeader, { childList: true });
}
```
All session metadata will be available via `window.ROR_METADATA` property. To get your custom logo, just use `window.ROR_METADATA.newLogo` value. In the example above, after login in as a user with username `admin` you will see a custom logo.
The whole example is a little complex but seems, kibana logo is also a loading indicator, we need to detect the loading state, replace the logo with a spinner, and after the loading, back the custom logo again.
