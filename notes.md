# FeM Web Security

- Course materials - https://github.com/mike-works/web-security-fundamentals

- To run class examples

  - Run local app at http://localhost:8080 - updated in index.js
  - Modern.ie - setup a VM with IE9 running - not sure this is needed

## Introduction

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
