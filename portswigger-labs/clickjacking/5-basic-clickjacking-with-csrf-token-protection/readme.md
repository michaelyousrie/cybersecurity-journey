# Basic Clickjacking with CSRF Token Protection

[Lab Link](https://portswigger.net/web-security/learning-paths/clickjacking/clickjacking-how-to-construct-a-basic-clickjacking-attack/clickjacking/lab-basic-csrf-protected)

**Difficulty:** Apprentice

**Objective:** Craft a decoy HTML page that frames the target application's account page and tricks the user into clicking the "Delete account" button, despite CSRF token protection.

This lab is best watched on the YouTube video where we go through the clickjacking concepts and then solve the lab in real time.

[YouTube Video](https://youtu.be/nL4p-wFvL5o)

## Lab Overview

The target application has a "Delete account" button on the user's account page. This button is protected by a CSRF token. Normally, CSRF tokens prevent attackers from forging requests on behalf of users. But clickjacking doesn't forge requests. It tricks users into making the request themselves.

The victim will click on elements displaying the word "click" on a decoy website. Our job is to build that decoy page with a hidden iframe underneath that aligns the real "Delete account" button exactly where the victim clicks.

**Login credentials provided:** `wiener:peter`

## Why CSRF Tokens Don't Help Here

CSRF tokens work by ensuring that a request came from the legitimate application, not from a forged form on an attacker's site. But in a clickjacking attack, the user IS interacting with the legitimate application. The real page is loaded in an iframe, the real button exists, and the real CSRF token is present. The user's browser sends everything correctly because from the browser's perspective, this is a genuine user action.

The only thing that's fake is what the user sees on screen.

## The Exploit

### Step 1: Understand the Target

Log in with `wiener:peter` and visit the account page. There's a "Delete account" button. Clicking it deletes the account. The form includes a CSRF token.

### Step 2: Build the Decoy Page

Create an HTML page that:
1. Loads the target's account page in a transparent iframe
2. Positions a visible "Click me" button directly over where the "Delete account" button appears in the iframe
3. Makes the iframe invisible so the victim only sees the decoy

The core HTML structure is in the `./decoy.html` file included in the same directory here.

### Step 3: Deliver the Exploit

Host this HTML on the exploit server provided by the lab. When the victim visits the page, they see "Click me" and click it. Their click passes through to the invisible iframe and hits the real "Delete account" button. The CSRF token is included automatically because the legitimate page is loaded in the iframe.

The account is deleted. Lab solved.

## How the Attack Works Visually

What the victim sees:

    +---------------------------+
    |                           |
    |     [Click me]            |
    |                           |
    +---------------------------+

What actually exists on the page:

    +---------------------------+
    |  (invisible iframe)       |
    |     [Delete account]      |  <-- real button, invisible
    |                           |
    +---------------------------+
    |     [Click me]            |  <-- decoy text, visible but behind iframe
    +---------------------------+

The victim clicks "Click me" but their click lands on "Delete account" in the invisible iframe above it.

## Key Takeaways

- **CSRF tokens do not prevent clickjacking.** They prevent forged requests, but clickjacking uses real requests made by real users on the real page. The token is present and valid because the legitimate page is loaded.

- **X-Frame-Options is the defense.** Setting `X-Frame-Options: DENY` or `X-Frame-Options: SAMEORIGIN` prevents the page from being loaded in an iframe at all, killing the attack at its root.

- **Content-Security-Policy frame-ancestors is the modern defense.** `Content-Security-Policy: frame-ancestors 'none'` achieves the same result as X-Frame-Options but with more flexibility and is the recommended approach.

- **Opacity doesn't mean invisible to the browser.** An iframe with opacity 0.0001 is effectively invisible to humans but the browser still renders it and processes clicks on it normally.

- **Positioning is everything.** The attack only works if the decoy element aligns precisely with the target button. Getting the pixel positioning right is the main technical challenge.
