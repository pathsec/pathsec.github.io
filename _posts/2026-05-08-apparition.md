---
layout: post
title: "Apparition: Browser-in-the-Middle Phishing"
subtitle: "Stealing firefox profiles."
date: 2026-05-08
tags: [red-team, phishing, tooling, docker]
---

Last year Mandiant published a blog post on Browser-in-the-Middle (BitM) phishing. Several of the resources they mentioned as useful in their research, I also found to be helpful:

- [https://cloud.google.com/blog/topics/threat-intelligence/session-stealing-browser-in-the-middle](https://cloud.google.com/blog/topics/threat-intelligence/session-stealing-browser-in-the-middle)
- [https://mrd0x.com/bypass-2fa-using-novnc/](https://mrd0x.com/bypass-2fa-using-novnc/)
- [https://link.springer.com/article/10.1007/s10207-021-00548-5](https://link.springer.com/article/10.1007/s10207-021-00548-5)
- [https://fhlipzero.io/blogs/6_noVNC/noVNC.html](https://fhlipzero.io/blogs/6_noVNC/noVNC.html)
- [https://github.com/JoelGMSec/EvilnoVNC](https://github.com/JoelGMSec/EvilnoVNC)
- [https://github.com/fkasler/cuddlephish](https://github.com/fkasler/cuddlephish)

[Apparition](https://github.com/pathsec/Apparition) is my take on BitM that I built to closely resemble the design described by Mandiant. It delivers isolated Firefox sessions to targets via a unique link, streams a browser to the victim over noVNC, and captures the full Firefox profile upon the user logging into the targeted service.

## What is BitM?

Browser-in-the-Middle is a phishing technique where the attacker doesn't intercept credentials, they intercept the browser session itself. Instead of cloning a login page, you serve the target a real browser running on your infrastructure pointed at the real login page.

## How Apparition Works

There are two primary components: a control server and the noVNC containers.

**Control Server** is a Node.js/Express application. It manages administration, campaigns, generates invite tokens, handles session lifecycle, and orchestrates the necessary Docker operations.

**noVNC Containers** are ephemeral Docker containers, one per session, each running an XFCE desktop, Firefox, and noVNC. When a target visits `/join/<token>`, the control server spins up a container, assigns it a port in the 6900–6999 range, and forwards the WebSocket stream to the target's browser. From the target's perspective, they're looking at a normal browser window.

## How a user is phished

Right now Apparition offers two ways to define when a firefox profile will be exported and available to the attacker.

**Cookie-based:** You specify a cookie name and domain to watch. Once that cookie appears in the browser (typically after a successful login), Apparition considers the session complete and archives the Firefox profile.

**URL-based:** You specify a URL pattern. When Firefox navigates to a matching URL, a post-login redirect, a dashboard landing page, a confirmation URL — the Firefox profile is archived.

## What You Get Out of a Session

Depending on what the target did during their session, the exported firefox profile can contain:

- Authenticated session cookies
- Saved form data and autofilled credentials
- Browsing history and cached responses
- Any files downloaded during the session

## Considerations, Limitations & Closing

Apparition mounts the Docker socket into the control server container to manage sibling containers. In production, restrict this with [docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy) rather than exposing the full socket. 

I have done my best to design this with a security-mindset for the infrastructure but have not fully explored the idea of an adversary performing container breakout and attempting to interact with the administrative server.

Cloudflare Turnstile integration is available if you need bot protection on the join endpoint.

You may run into scalability issues if trying to run this on, for example, a low-powered EC2. The noVNC sessions only live for so long, however if you have multiple victims being phished at the same time you may run into resource availability issues. Keep in mind the scope when selecting your hosting environment. At some point I'd like to optimize this but for now it is absolutely a consideration you should have.

If you have contributions or improvements, or notice an issue, pull requests are open: [https://github.com/pathsec/Apparition](https://github.com/pathsec/Apparition)
