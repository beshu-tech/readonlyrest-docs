---
description: Custom middleware
---

# Custom middleware

Sometimes, Enterprise users might need more flexibility and customize the plugin behavior to adjust the product to the business needs.
There are two options to declare the custom middleware:
- JS file: `readonlyrest_kbn.custom_middleware_inject_file: '/path/to/your/file.js'` // You can also use a relative path here. It's relative to the Kibana root folder
- Inline: `readonlyrest_kbn.custom_middleware_inject: 'function test(req, res, next) {logger.debug("custom middleware called"); next()}'`
