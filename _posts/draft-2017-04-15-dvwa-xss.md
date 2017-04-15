---
layout: post
section-type: post
title: Exploiting Cross-site scripting vulnerabilities
category: tech
tags: [ 'redteam', 'kali', 'dvwa', 'grabber' ]
---

What is Cross-site Scripting (XSS) and why is it so important?
XSS enables the injection of client-side instructions into a web applications that is viewed by other users.
In case you're wondering what can go wrong with that, just think about an attacker grabbing your session id, which will
enable a session-hijacking, which effectively means stealing your identity in the web application.
This, in combination with the XSS vulnerabilities being at the top 3 of the [OWASP Top Ten](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project#tab=OWASP_Top_10_for_2017_Release_Candidate) web application vulnerabilities, makes this kind of attacks crucial.

The anatomy of the attack is very simple.
Consider any user input as an attack surface, where we can enter javascript code in it.
If the web application returns this input to the client without encoding the '<' and '>' characters then the browser will interpret the script tags as javascript instructions sent by the web application and it will execute it.
Now, we can get access to the web page's cookie:


<pre><code data-trim class="javascript">

<script>alert(document.cookie)</script>

</code></pre>

http://127.0.0.1/dvwa/vulnerabilities/xss_r/?name=%3Cscript%3Ealert%28%27Oops!%27%29%3C%2Fscript%3E

127.0.0.1/dvwa/vulnerabilities/xss_r/?name=%3Cscript%3Ealert(%27Oops!%27)%3C%2Fscript%3E
