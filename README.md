 xss-reflected-writeup
PortSwigger lab – Reflected XSS vulnerability analysis
Summary
While testing the search functionality of the application, I found that user input is reflected back into the webpage without proper sanitization. By injecting JavaScript code, it was possible to execute arbitrary scripts in the victim’s browser.

 Recon
I interacted with the search feature of the website.

When I searched for a value like:
test

The page displayed:
"You searched for: test"

This indicated that my input was being returned directly in the response page.

This is a potential vulnerability because browsers execute JavaScript inside HTML pages.

---

 Intercepted Request
I intercepted the request using Burp Suite Proxy.

GET /?search=test HTTP/1.1
Host: target

The `search` parameter contained my input.

---

 Testing
Since the application reflected my input, I attempted to inject JavaScript.

Payload used:
<script>alert(1)</script>

Modified request:

GET /?search=<script>alert(1)</script> HTTP/1.1
Host: target

---

 Result
After forwarding the request, the browser executed the script and displayed an alert popup.

This confirmed the application is vulnerable to Reflected Cross-Site Scripting.

---

 Root Cause
The application inserts user-controlled input directly into the HTML response without encoding.

Server assumption:
Input = text

Browser interpretation:
Input = executable JavaScript

Because the output was not encoded, the browser executed attacker-supplied code.

---

 Impact
An attacker can:
- Steal session cookies
- Hijack user accounts
- Redirect users to phishing websites
- Perform actions as the victim

Example attack:
A victim clicks a crafted link containing malicious JavaScript and their session can be stolen.

---

 Remediation
The application should never render raw user input in HTML.

Proper fixes:
- Output encoding (HTML encode special characters)
- Input validation
- Use secure templating frameworks
- Implement Content Security Policy (CSP)

Example safe output:
<script> becomes &lt;script&gt;

The browser will treat it as text instead of executing it.
