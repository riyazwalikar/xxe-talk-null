# XML External Entity (XXE) Attacks

Riyaz Walikar | @riyazwalikar | @wincmdfu



## About Me

- Riyaz Walikar | @riyazwalikar | @wincmdfu
- Chief Offensive Security Officer at Appsecco (@appseccouk)
- Blog at https://blog.appsecco.com and https://ibreak.software




## What are XXE attacks?

- A type of XML Injection attack in an application that parses XML data
- XML parser expands the Entity declaration within a DTD submitted as part of the XML data
- The Entity expansion could point to various external resources based on the Entity attribute and URI scheme used
    - For example, `file://` would point to a local file
- Listed at A4 in OWASP Top 10 2017



## Conditions for XXE?

- user input should be parsed directly (without sanitisation) by the XML parser
- the XML parser should perform external entity expansion
- the system should allow requests to be made to external systems (if pointing to resources outside the system)



## Sample XML data and Entity

```
    <products>
        <product>
            <name>Television</name>
            <code>100</code>
            <tags>entertainment</tags>
            <description>This is a pretty cool TV!</description>
        </product>
    </products>
```

```
    <!DOCTYPE test [<!ENTITY desc "This is a pretty cool TV!">]>
    <products>
        <product>
            <name>Television</name>
            <code>100</code>
            <tags>entertainment</tags>
            <description>&desc;</description>
        </product>
    </products>
```



## Where to find XXE?

- Apps that support XML transports. Examples include:
    - An app that supports XML data transfer via a Form HTTP POST
    - An app that allows for Microsoft Office Document uploads
    - An app that sends XML data via a UDP packet to a server that processes the packet
    - Apps that rely on SAML for authentication. SAML uses XML for identity assertions and may be vuln



## How to detect XXE?

- As XXE is a subclass of XML Injection attacks, detect if application is vulnerable to XML Injection first
- If the server parses unclosed tags (<,>) to return either no data, error messages or 500 Server Status
- Sometimes the server may not reveal that the XML parser choked on the malformed XML
    - Send sample XML data with Entity pointing to a server you control
    - This would be a Blind XXE Vulnerability


### Quick steps:
(Use Burp/favorite interception proxy to do this)
- Send unclosed tags in the parameter being tested
- Send closed tags to see if there is a difference in output
- Send valid XML input that the application is expecting
- Send XML containing an ENTITY element with either the SYSTEM or PUBLIC attribute pointing to a resource


### Sample XXE Payload with SYSTEM

    <!DOCTYPE test [<!ENTITY abc SYSTEM "file:///etc/passwd">]>
    <products>
        <product>
            <name>Television</name>
            <code>100</code>
            <tags>entertainment</tags>
            <description>&abc;</description>
        </product>
    </products>


### Sample XXE Payload with PUBLIC

    <!DOCTYPE test [<!ENTITY abc PUBLIC "file" "file:///etc/passwd">]>
    <products>
        <product>
            <name>Chicken</name>
            <code>101</code>
            <tags>tasty food</tags>
            <description>&abc;</description>
        </product>
    </products>



## Demo



## What can be done with XXE?

<ul align="left">
<li class="fragment">
Read local files like `passwd` or `shadow` (if server is running as root)
</li>
<li class="fragment">
Access internal applications not visible from the Internet (SSRF)
</li>
<li class="fragment">
Port scan or attack services on arbitrary ports within the internal network or the Internet (XSPA)
</li>
<li class="fragment">
Under certain conditions, remote code execution may also be possible
</li>
<li class="fragment">
Perform a Denial of Service attack using recursive expansion or reading a file like `/dev/random`
</li>



## Prevention

- Input validation
- Disable External Entity Expansion using application code and app server configuration (see references)
- Use egress filtering to prevent the system with the XML parser from making outbound connections



## References

- [XML Entities][1]
- [OWASP Top 10 A4 - XXE][2]
- [Damn Vulnerable Node Application][3]
- [Different URI schemes][4]
- [Detecting and Exploiting XXE in SAML][5]
- [Billion Laughs Recursive Expansion Attack][6]
- [XXE OWASP Prevention Cheat Sheet][7]

[1]: https://www.w3resource.com/xml/entities.php
[2]: https://www.owasp.org/index.php/Top_10-2017_A4-XML_External_Entities_(XXE)
[3]: https://github.com/appsecco/dvna
[4]: https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml
[5]: https://web-in-security.blogspot.com/2014/11/detecting-and-exploiting-xxe-in-saml.html
[6]: https://en.wikipedia.org/wiki/Billion_laughs
[7]: https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Prevention_Cheat_Sheet