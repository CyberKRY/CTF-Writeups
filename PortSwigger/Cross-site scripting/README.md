<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS.webp" alt="XXS" width="800" />
</p>

<h1>What is Cross-Site Scripting (XSS)?</h1>

Cross-Site Scripting (XSS) is a class of web vulnerability where an application includes attacker-controlled data in web pages without sufficient validation or escaping, allowing malicious scripts to run in other users’ browsers. XSS undermines user trust and can lead to account takeover, data theft, session hijacking, and phishing.

<h2>Main types of XSS</h2>

Reflected XSS (non-persistent) — payload is included in a request (URL, form) and reflected back in the response. Triggered when a victim follows a crafted link or submits data.

Stored XSS (persistent) — attacker data is saved on the server (e.g., in a database, comment, profile) and later rendered to other users, causing repeated exploitation.

DOM-based XSS — vulnerability occurs entirely in the browser when client-side scripts use untrusted data (e.g., location, document.cookie) to modify the DOM without proper sanitization.

<h2>Contexts and payload considerations</h2>

XSS payloads depend on context: HTML body, attributes, event handlers, JavaScript contexts, CSS, or URL/JS constructors. Effective testing examines all injection sinks and contexts (innerHTML, eval(), innerText vs textContent, document.write, attributes, etc.).

<b>Impact</b>

* Steal cookies, tokens, or local storage

* Keylogging and form capture (credential harvesting)

* Performing actions on behalf of users (CSRF amplification)

* Delivering malware or phishing content

* Bypassing MFA in some flows
