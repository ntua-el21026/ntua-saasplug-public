# Course provider interfaces

The three hypothetical course providers are redPlug, greenPlug, and bluePlug. The original course PDFs, OpenAPI YAML files, and Postman exports describe request and response shapes and supported adapter development and manual contract comparisons. Obtain those materials through [Helios](https://helios.ntua.gr/course/view.php?id=1015) or consult the [hosted Swagger access guide](swagger_apis_hypothetical_providers.md). Provider access requires the NTUA network or VPN and authorized credentials.

The [service tree](../../../services/README.md) contains all three adapters and their payload-normalization logic. The project-authored [OpenAPI contract](../../../openapi.json) and [Postman suites](../../postman/README.md) describe the saasPlug interface. Code inspection and offline checks can run against local fixtures. Live-provider reproduction requires matching contracts and valid test keys.
