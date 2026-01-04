---
layout: default
title: How to make chrome extensions safer by blocking their Internet access
nav_order: 2
feed: true
date: 2025-12-23
---

# How to make chrome extensions safer by blocking their Internet access
{: .no_toc }


{: .fs-6 .fw-300 }

{% if page.date %}
  <time datetime="{{ page.date | date_to_xmlschema }}">
    {{ page.date | date: "%Y-%m-%d" }}
  </time>
{% endif %}

# Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# The security issues for Chrome/Chromium extensions
Browser extensions are powerful—and that power cuts both ways.

Over the years, there have been **many documented cases of Chrome extensions turning malicious**. Some were malicious from the start; many more followed a more troubling pattern:
they were **perfectly legitimate at first**, built trust, gained users, and then **became malicious after an update**.

See <https://www.techradar.com/pro/security/this-google-chrome-extension-has-been-silently-stealing-every-ai-prompt-its-users-enter>  and  <https://thehackernews.com/2025/12/shadypanda-turns-popular-browser.html>.


## A recurring pattern: trust first, exploit later

A common story looks like this:

1. An extension provides a genuinely useful feature
2. It gains thousands or millions of users
3. Ownership is sold, or the maintainer’s goals change
4. A “normal” update is pushed
5. Suddenly, the extension:

   * Exfiltrates browsing data
   * Injects ads
   * Tracks users
   * Communicates with remote servers without transparency

Because Chrome extensions **auto-update by default**, users often never notice when the behavior changes.

Even worse:
many malicious extensions technically follow Chrome’s permission model, meaning they aren’t blocked automatically.


## Why Internet access is the real risk

Most damage caused by malicious extensions requires **network access**:

* Uploading browsing history
* Sending fingerprints or cookies
* Fetching remote scripts
* Receiving commands from a control server
* Injecting ads or redirects dynamically

Without Internet access, an extension is dramatically limited:

* It can’t leak data externally
* It can’t download new malicious payloads
* It can’t phone home
* Its behavior becomes **auditable and deterministic**

In other words:

> **An extension without Internet access can only do what you can see.**



## Many extensions don’t actually need the Internet

A surprising number of popular extensions:

* Modify page behavior
* Change UI
* Block elements
* Add keyboard shortcuts
* Manipulate DOM content

All of these can work **entirely offline**.

Yet developers often request:

* `https://*/*`
* `http://*/*`
* Broad `connect-src` permissions

This creates unnecessary risk.

## Blocking Internet access as a defensive baseline

Blocking Internet access for extensions should be treated like:

* Running apps behind a firewall
* Sandboxing untrusted code
* Applying least-privilege principles

By default:

* Extensions should **not** be allowed to talk to the Internet
* Exceptions should be explicit, minimal, and auditable

This flips the trust model:

* From *“trust until proven malicious”*
* To *“deny until proven necessary”*



## The bigger picture: defense against future updates

Even if you trust an extension **today**, you don’t control:

* Future updates
* Ownership changes
* Monetization pressure
* Supply-chain attacks

Blocking Internet access protects you not just from known threats, but from **unknown future ones**.

It’s a safeguard against:

* Silent behavior changes
* Malicious updates
* Compromised maintainers


# How to defend your browser against potentially malicious extensions
The most straightforward idea is to block Internet access for extensions. However, Chrome itself does not provide a way for users to do this. 

Luckily, you could choose another approach: copy the files for Chrome extensions, modify their `manifest.json` by addint the following snippet to block the Internet access

```
"extension_pages": (
        "default-src 'self';object-src 'self';connect-src 'self';style-src 'self' 'unsafe-inline'")
```

Then, you need to enable the developer mode on the extensions page of Chrome.

Manually modifying the `manifest.json` files for all extensions seems a daunting task. Besides, you also need to manually download the new versions of extensions, extract them, and modify the `manifest.json` again to keep the extensions updated.

Therefore, I've written some simple Python scripts for this task <https://github.com/diqbpow3c/ChromeExtensionHardener>.


# How does it work
The script `secure_chrome_extension_manager.py` modifies the `manifest.json` of extensions, blocking Internet access by adding the aforementioned content security policy configuration:
```
"extension_pages": (
        "default-src 'self';object-src 'self';connect-src 'self';style-src 'self' 'unsafe-inline'")
```
The content security policy for extensions requiring only LAN access is similar.

The script downloads and unpacks the extensions to local folders, modifies the manifest.json files by adding the above CSP.

# How to use 
Find out the extension IDs of Chrome extensions (chrome://extensions/), backup config files, and remove them from your browser. Turn on the developer mode in the extensions page of Chrome.

Clone the repo, and install requirements by
`pip install crx3 plyer`.

Modify the config file `extensions.yaml`. Put the extension ID of extensions to the `extensions` section, and optionally put the IDs of extensions that require only LAN access to the `extensions_requiring_lan_access` section.


`python secure_chrome_extension_manager.py`  

You can use cron (Linux) or Task Scheduler (Windows) to periodically run the above command, so that updates to the extensions are automatically handled.

Go to the extensions page of Chrome, load unpacked extensions.

**Attention**: after updating extensions, you need to go to the extensions page again and reload the extensions. You can also use extensions like [chrome-extensions-reloader](https://github.com/arikw/chrome-extensions-reloader) to simplify this task.

**Caution**: This code is not fully tested, but works on my computers and extensions. Use it with caution.


