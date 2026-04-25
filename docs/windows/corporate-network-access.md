# Corporate Network Access and Claude Installation

Some enterprise networks use web filtering software (Zscaler, Forcepoint, Cisco Umbrella, etc.) that blocks `claude.com/download` under categories like "Developer Tools" or "AI/ML Applications". This is a network policy issue, not a Superpowers or Claude bug.

## The Problem

When your organization's web filter blocks `claude.com/download`, you'll typically see a block page from your filter vendor instead of the download starting. The Claude desktop app installer (`Claude Setup.exe` on Windows) cannot be downloaded through that network connection.

## Solutions

### Option 1: Request IT whitelisting

Submit a request to your IT helpdesk to whitelist `claude.com` (and `claude.ai` if also blocked). Most corporate filter products let users request access to blocked categories or specific URLs. Look for a "Request Access" link on the block page itself — many vendors include one.

What to ask for:
- Unblock `claude.com` and `claude.ai`
- Or allow the "Developer Tools" / "AI/ML Applications" category for your user account

### Option 2: Use Claude in the browser

If `claude.ai` is not blocked on your network, you can use Claude directly in a web browser without installing the desktop app. Most Superpowers workflows work through Claude's web interface.

### Option 3: Download on a personal network

If you have a personal mobile hotspot or can access a home network, download the installer there and transfer it to your work machine. The installer does not require network access to `claude.com` during installation itself — only the initial download is blocked.

## This Is Not a Superpowers Issue

Superpowers is a plugin that runs inside Claude. If you cannot download or access Claude at all, Superpowers cannot help — the underlying platform needs to be reachable first. File network access issues with your IT department, not here.
