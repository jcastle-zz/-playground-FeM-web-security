# FeM Web Security

- Course materials - https://github.com/mike-works/web-security-fundamentals

- To run class examples

  - Run local app at http://localhost:8080 - updated in index.js
  - Modern.ie - setup a VM with IE9 running - not sure this is needed

## Introduction

- Client-side security - XSS (e.g., injection - code, SQL), CSRF (e.g., authentication requests), clickjacking (e.g., iFrame duplicates, URI redress attack), third party assets (e.g., dependencies)
- Server-side security - Man-in-the-Middle, TLS(HTTPS), HTTPS Downgrade

- Problems with frontend development and security

  - Have to consider features and deadlines vs. security
  - Web developers have fallen behind backend and devops
  - Attacks are escalating in severity
  - Barriers to staging an attack are lower than ever

- Types of hackers

  - Black hat - Causes damage, hold data for ransom. Driven by personal gain or desire to do as much damage as possible (because they can).
  - Grey hat - Break into systems but _usually cause no damage_. Mostly driven by curiosity. Sometimes report vulnerabilities they find.
  - White hat - Break into systems _with permission_ and responsibly disclose anything they find. Make money from bug bounties and consulting as penetration testers.

- Zero day bug - but no one knows about to get a foothold of a system. Begins the race to patch vs try to exploit it.

- Hacker motives

  - Gather information about your system
  - Research vulnerabilities
  - Get a foothold in the system
  - Use that foothold to escalate to more serious attacks

## Cross-Site Scripting (XSS)

- Definition

  - An injection attack - putting content in a place that is designed for text, trick a system to treat as code and execute it
  - Vulnerabilities are prevalent - difficult to pin down how many sites have this issue but very high, more than 30% of sites are vulnerable to this, some browsers can execute some code some of the time
  - Allow attacker to read data or perform operations on user's behalf - can result in full system control, simulate admin users, drop tables

- XSS Categories

  - Stored XSS - Code that executes attacker's script is persisted. Adding something to a DB - registering user name with code in it.
  - Reflected XSS - Transient response from server causes script to execute (i.e., validation error). Can trick temporary response from server like a validation message.
  - DOM based XSS - No server involvement is required (i.e., pass code in via queryParams). Query param escaping into the DOM.
  - Blind XSS - Exploits vulnerability in another app (i.e., log reader), the attacker can't see or access under normal means. Sub-category of XSS. Using logging from an external app into an internal app. Vulnerability comes with sucking public data into an internal app. Often need credentials b/c behind firewall.

- XSS Danger Zones

  - User generated rich text (i.e., WYSIWYG) - drop an image tag, start experimenting
  - Embedded content - drop an iframe, put an object
  - Anywhere users have control over a URL - older browsers can have javascript: followed by code
  - Anywhere user input is reflected back (i.e., "couldn't find xyz")
  - Query parameters rendered into DOM
  - element.innerHTML = ? - could be arbitrary HTML

- XSS Questions for Web Developers

  - How confident are you in the XSS protection of your OSS libraries? - One thing to look for is a procedure for fixing vulnerability issues, they have an email and team, GH issue should not be the way to report vulnerabilities.
  - How carefully do people scrutinize browser plugins (i.e., Chrome extensions)? - There is skeptical and there is vigilant. Check for scope of permissions. Use incognito tab for banking and other sensitive browsing and don't let extensions work on incognito tab.
  - If XSS happens, what's your exposure? - Reasons to restrict what can happen through UI and API.
  - In your apps, what could a successful XSS attack escalate to? - Examples include access to data, access to files.

- XSS defenses: Never put untrusted data in these places
  _don't ever trust user data_

  - Directly in a script
  - In an HTML comment
  - In an attribute name
  - In a tag name
  - Directly in a <style> block

- XSS defenses: Escape data before putting it into HTML

  1. Sanitize data before it is persisted
  2. Sanitize data before it is presented on the screen

  - Usually want to do both, always holes in sanitation methods
  - Just about every view library does this automatically (e.g., React, View, Ember, etc.)

- Content Security Policy (CSP) - https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP

  - For HTTP headers to check for XSS attempts
  - Can use Helmet.js - https://helmetjs.github.io/ for providing middleware checks for Express and others - for this example, goes into ./server/index.js
  - Browsers can't tell the difference between scripts downloaded from your origin vs another. It is a single execution context.
  - CSP allows us to tell modern browsers which sources they should trust, and for what types of resources
  - This information comes via a HTTP response header or meta tag
  - Multiple directives are separated by semicolon
  - Re-defining a directive with the same name has no effect
  - By default, directives are permissive
  - _Look at slides for a selection of CSP directives_

- Malicious Attachments

  - Malicious content can be included in attachments
  - Can add malicious content to images. There is a part of the image that is not seen (e.g., metadata pertaining to the image)
  - Example in class - used a jpg attachment to insert code onto an HTML page 😩

- Stopping Malicious Attachments

  - Be restrictive on file upload types
  - Don't trust mime types
  - Don't trust extensions
  - More files are accepted, it allows a drop site for users, becomes wasteland of XSS hell
  - Compress images, it will drop all non-visible data (tool to use Image Magic)
  - Research attachment types thoroughly - pdfs allow embedded code (e.g., JS)

## Cross-Site Request Forgery (CSRF)

- Takes advantage of the fact that cookies (or Basic Authentication credentials) are passed along with requests
- One of several good reasons to align with REST conventions
- You know if you are vulnerable if your server looks to a cookie sent along with the request or basic authentication credentials sent along with the request in order to authenticate or authorize a user. Authenticate = this is who you are, Authorize = do you have the proper privileges.

Three defenses:

1. CSRF Tokens

- Only Basic or cookie authentication schemas are vulnerable
- Exception: "Client side cookie"
- Key concept: using cookies doesn't require the ability to read cookies
- localStorage/sessionStorage - alternatives that don't have this problem - only accessible on client side
- One way to solve CSRF is to pass a token. These tokens change with each request in an unpredictable way. These tokens are disposable.
- The token value is not in the cookie that someone can see/use
- Kind of like 2FA because it provides for authentication and knows the request is from a trusted place/source
- For server rendered apps: meta tags are fine

2. Request Origin

- Another best practice is to validate the request origin
- Modern browsers send an Origin header which cannot be altered by client-side code, with each request (IE11 does not in some cases)
- In cases where there is no Origin header, there's almost always a Referer header
- When behind a proxy, you can usually get some information from Host and X-Forwarded-Host headers

3. Cross-Origin Resource Sharing (CORS)

- This is what permits browsers to send a request from one domain to another
- A preflight OPTIONS request gives server a chance to indicated what's allowed, main request follows

## Clickjacking

- Trick the user into performing an action they don't see
- Usually occurs through an elaborate positioning of iframes
- These are UI redress attacks
- Can be used to capture keystrokes, really refined clickjacks create illusion that user is clicking on what they want, or typing where they want
- Comprimising the user and not the system

- Stopping Clickjacking

  - X-Frame-Options - DENY, SAMEORIGIN, ALLOW-FROM [url]
  - Chrome/Safari don't respect allow-from. Use frame-ancestors CSP directive instead
  - This applies to the TOP LEVEL frame
  - Slides include <style id="clickjack">

## Third Party Assets

Three types of third party assets:

1. Pulling in resource someone else is hosting like with a CDN
2. Using a version dependency (e.g., NPM), pulling down source code from some repository - package-lock.json and yarn-lock support keeping dependencies set for the build
3. The worst type, inserting scripts or HTML into application

All are third party because we didn't write the code, we probably didn't thoroughly review the code, and we are not hosting the code.

Issues with third party assets:

- The people who write your dependencies make mistakes
- Recommendations:

  1. Reproducible builds, with a lockfile
  2. Use LTS (Long Term Support aka stable) versions where you care less about bleeding edge features
  3. Support _bug bounties_ in important OSS projects
  4. Run tests that assert only expected requests are sent out (e.g., in acceptance tests)

- For situations where using vendor tags (i.e., with Google Analytics):

  - These can be updated independently of deployments
  - Definitely avoid adding scripts that add more scripts
  - When "fail secure" is desired, add your own SRI (Subresource Integrity Fallback) to the script tags - for reference: https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity
  - Ask that your vendors VERSION scripts, so there is control when new code lands (and the SRI doesn't break)

## Man-in-the-Middle

- Definition: cyber attack where the attacker secretly relays and possible alters the communications between two parties - https://en.wikipedia.org/wiki/Man-in-the-middle_attack
  - Hacker can eavesdrop and tamper with communication between sender and server
  - XSS at will
  - Capture user credentials and be malicious - try credentials on other sites
- DNS (Domain Name Service) - translates hostname into IP address

Defense:

- Encrypt data in-flight with HTTPS
  - Use TLS (Transport Layer Security)
  - Use a secret key to read or alter request/response
  - Certificates identify domains and require domain validation
  - Enhance validation often requires government ID but that's just issuer policy

* Any certificate with matched domain name will work for HTTPS so anyone with cert can get HTTPS connection to the domain

## HTTPS

- 56% of web uses HTTPS (might be more now, rest using HTTP)
- Definition from Wikipedia: The principal motivations for HTTPS are authentication of the accessed website, and protection of the privacy and integrity of the exchanged data while in transit. It protects against man-in-the-middle attacks, and the bidirectional encryption of communications between a client and server protects the communications against eavesdropping and tampering. In practice, this provides a reasonable assurance that one is communicating with the intended website without interference from attackers.
- Definition URL on web: https://en.wikipedia.org/wiki/HTTPS#:~:text=Hypertext%20Transfer%20Protocol%20Secure%20(HTTPS,Hypertext%20Transfer%20Protocol%20(HTTP).&text=In%20HTTPS%2C%20the%20communication%20protocol,TLS%2C%20or%20HTTP%20over%20SSL

- HTTPS: Cryptography

  - Two types of encryption involved: Symmetric encryption and Public Key encryption (i.e., public for writing & private key for reading) - popular algorithm is RSA
  - Symmetric is faster, has no practical limit on size of content
  - Keys generated on per connection basis
  - One catch is the safety of the key

- TLS Handshake

  - Passing of certificates and session key
  - Allows for encrypted communication
  - Public and private keys are just for the key exchange, all other stuff (i.e., to transfer as in content, data) done by symmetric encryption because it's much faster

- OpenSSL

  - Industry standard library for crypto
  - Don't implement your own algorithm, handshake, protocol, etc. - there are scripts for this in the slides, but wouldn't want to go this route
  - OpenSSL is not user friendly
  - You won't need it all that often

## HTTPS Downgrade
