Other than using the latest XML libraries, certain XML configurations for web applications can help reduce the possibility of XXE exploitation. These include:

- Disable referencing custom `Document Type Definitions (DTDs)`
- Disable referencing `External XML Entities`
- Disable `Parameter Entity` processing
- Disable support for `XInclude`
- Prevent `Entity Reference Loops`
Error-based XXE exploitation.  we should always have proper exception handling in our web applications and `should always disable displaying runtime errors in web servers`.
## avoid xml
With the various issues and vulnerabilities introduced by XML data, many also recommend `using other formats, such as JSON or YAML`. This also includes avoiding API standards that rely on XML (e.g., SOAP) and using JSON-based APIs instead (e.g., REST).

## WAF
using Web Application Firewalls (WAFs) is another layer of protection against XXE exploitation. However, we should never entirely rely on WAFs and leave the back-end vulnerable, as WAFs can always be bypassed.