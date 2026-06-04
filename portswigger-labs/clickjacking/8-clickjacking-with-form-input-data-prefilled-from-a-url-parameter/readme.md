# Clickjacking with Form Input Data Prefilled from a URL Parameter

[Lab Link](https://portswigger.net/web-security/learning-paths/clickjacking/clickjacking-clickjacking-with-prefilled-form-input/clickjacking/lab-prefilled-form-input)

**Difficulty:** Apprentice

**Objective:** Craft a decoy HTML page that frames the account page with a prefilled email address and tricks the victim into clicking the "Update email" button.

This lab is best watched on the YouTube video where we go through the concepts and solve the lab in real time.

[YouTube Video](https://youtu.be/FFXFTZ91_-8)

## Lab Overview

This lab extends the basic clickjacking technique. The target application has an "Update email" form on the account page. The form's email field can be prefilled using a URL parameter. By combining this with a clickjacking iframe, we can control what email address gets submitted when the victim clicks the hidden button.

**Login credentials provided:** `wiener:peter`

## What Makes This Different from Basic Clickjacking

In the previous clickjacking lab, we tricked the victim into clicking an existing button with its existing data. Here, we need the victim to submit specific data that we control. The URL parameter prefill is what makes this possible. We load the account page with our attacker email already filled in, then overlay it with a decoy to trick the victim into clicking "Update email."

## The Exploit

### Step 1: Discover the Prefill Parameter

The account page's email update form accepts a URL parameter to prefill the email field. Loading the account page with a parameter like:

    /my-account?email=attacker@evil.com

Pre-populates the email field with our attacker-controlled address.

### Step 2: Build the Clickjacking Page

Using Burp Suite's Clickbandit tool, the process of creating the clickjacking proof-of-concept is significantly faster than crafting the HTML manually.

Clickbandit works by:
1. Loading the target page in a browser
2. Recording where you click
3. Generating a complete clickjacking HTML page with the iframe and decoy overlay properly positioned. We used Burpsuite's Clickbandit for this.

### Step 3: Deliver the Exploit

Host the HTML on the exploit server. When the victim visits the page:
1. The iframe loads the account page with the email already prefilled to our address
2. The victim sees only the "Click me" decoy
3. They click it, which actually hits the "Update email" button in the invisible iframe
4. The email is changed to our attacker-controlled address

Lab solved.

## Burp Suite Clickbandit

Clickbandit is a tool built into Burp Suite that automates clickjacking PoC creation. Instead of manually calculating pixel positions and writing HTML, you:

1. Open the Clickbandit tool in Burp
2. Navigate to the target page
3. Click the button you want the victim to click
4. Clickbandit generates the full exploit HTML

For penetration testing reports, this is invaluable. You can generate a working PoC in seconds rather than spending time on CSS positioning.

## Key Takeaways

- **URL parameter prefill combined with clickjacking is dangerous.** Individually, a prefillable form field is a minor convenience feature. Combined with clickjacking, it becomes an account takeover vector.

- **Clickbandit saves time.** For real-world pentests, manually crafting clickjacking HTML is unnecessary when Clickbandit can generate a working PoC automatically.

- **Frame-ancestors CSP is still the fix.** The same defense from the previous lab applies. Setting `Content-Security-Policy: frame-ancestors 'none'` prevents the page from being loaded in an iframe at all.

- **Don't accept sensitive form input from URL parameters.** If the email update form didn't accept prefilled values from the URL, this specific attack wouldn't work even without frame protection.

## Tools Used

- Burp Suite Community Edition
- Burp Suite Clickbandit
- PortSwigger Exploit Server