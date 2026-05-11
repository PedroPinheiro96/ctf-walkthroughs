# TryHeartMe – TryHackMe

- Difficulty: easy
- Date: 2026-05-09
- Tags: web, jwt, burp

## 🧩 Overview

```text
This challenge demonstrates how insecure handling of JSON Web Tokens (JWTs) can allow attackers to tamper with authentication data and escalate privileges.
```

## ⚙️ Tools Used

- [jwt.io/](https://www.jwt.io/)
- Burp Suite
- Developer Tools

## Walkthrough

First, I visited the homepage of the web server at `http://10.113.170.190:5000`.
The web server is hosting an ecommerce website and had 4 available products.

<div align="center"><img src="../attachments/Pasted image 20260509212145.png"/></div>

Attempting to purchase a product revealed that account credits were required and that authentication was necessary to complete purchases.

<div align="center"><img src="../attachments/Pasted image 20260509212157.png"/></div>

Then, I've started `Burp Suite` and added the domain to the `Scope` and started capturing requests.

<div align="center"><img src="../attachments/Pasted image 20260509212405.png"/></div>

With `Burp Suite` running, I created an account.

<div align="center"><img src="../attachments/Pasted image 20260509213542.png"/></div>

`Burp Suite` captured the request.
After successful registration, the application issued a `JWT-based session cookie` named `tryheartme_jwt`.

<div align="center"><img src="../attachments/Pasted image 20260509213639.png"/></div>

I visited the `Account` page and could see that I had `0` available credits.

<div align="center"><img src="../attachments/Pasted image 20260509215654.png"/></div>

The JWT was decoded using [jwt.io/](https://www.jwt.io/), revealing that the token stored authorization-related claims directly inside the payload, including:

- User role
- Credit balance

<div align="center"><img src="../attachments/Pasted image 20260509214007.png"/></div>

Because JWT payloads are only Base64-encoded and not encrypted, their contents can be modified if signature validation is improperly implemented.

I modified the token payload to:
- Change the role from `user` to `admin`
- Increase the available credits to `10000`

<div align="center"><img src="../attachments/Pasted image 20260509214033.png"/></div>

The modified `JWT` was then manually inserted into the browser session cookie using the browser developer tools.

<div align="center"><img src="../attachments/Pasted image 20260509214345.png"/></div>

After refreshing the session, I had now `10000` credits and could see the `Admin portal` button.

<div align="center"><img src="../attachments/Pasted image 20260509214356.png"/></div>

This demonstrated that server accepted the forged `JWT`. The application failed to properly validate JWT integrity and trusted client-supplied authorization data.

The `ValenFlag` product was now displayed on the `Shop` page too.

<div align="center"><img src="../attachments/Pasted image 20260509214408.png"/></div>

Finally, I clicked on the `ValenFlag` producted, and bought it

The flag was displayed:

<div align="center"><img src="../attachments/Pasted image 20260509214420.png"/></div>

## 🚩 Flag

```text
THM{v4l3nt1n3_jwt_c00k13_t4mp3r_4dm1n_sh0p} 
```

## 🧠 Lessons Learned

- JWT payloads are not encrypted and should never be trusted for authorization decisions without proper signature validation.
- Storing sensitive authorization data such as user roles and account balances inside client-controlled tokens increases the impact of token tampering vulnerabilities.
- Broken authentication mechanisms can allow attackers to escalate privileges without exploiting memory corruption or server-side code execution.
- Intercepting proxies such as Burp Suite are effective for analysing authentication workflows and session management mechanisms.

## 🛡️ Mitigation

- Always validate JWT signatures server-side before trusting token contents.
- Use strong secret keys and secure signing algorithms for JWT generation.
- Avoid storing sensitive authorization data such as account balances or privilege levels inside client-controlled tokens.
- Implement short token expiration times and token rotation mechanisms.
