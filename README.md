# Self-Managed Puppet Infrastructure Lab

A GitOps-driven Puppet environment I built and operate as a personal home lab — designed to mirror how production configuration-management systems are actually run.

This repo documents the architecture and decisions. The working control repo, hostnames, IP ranges, and credentials are intentionally kept private (see [Security Notes](#security-notes) below).

## Overview

The lab consists of a Puppet master managing both Linux and Windows agent nodes over a hardened, VPN-gated network, with a Git-based change workflow and a monitoring stack.

```
Git Repo (dev branch -> PR -> production)
  |
  v (merge to production)
Puppet Master (r10k_cron: scheduled r10k deploy, puppetserver, hiera-eyaml)
  |
  v (WireGuard tunnel)
Agent Nodes (Linux + Windows)
  |
  v
Zabbix Monitoring (LLD, custom checks, dashboards)
```

## What it does

* **GitOps workflow** - changes are made on the `dev` branch, reviewed via PR, and merged to `production`. There's no separate feature-branch-per-change step - everything lands on `dev` first, then goes through PR review before `production`.
* **Scheduled deployment** - the Puppet master runs a cron-driven r10k deploy (`r10k_cron`) that periodically pulls the `production` branch and updates the deployed environment. There's no CI/CD runner reaching into the lab network to push changes - the master pulls on its own schedule, keeping the deployment surface entirely inside the WireGuard-gated network.
* **Secrets management** - all sensitive data (keys, credentials) is encrypted at rest using hiera-eyaml. Nothing sensitive is ever committed in plaintext.
* **Cross-platform node management** - the same control repo manages both Linux hosts and a Windows Server node, with OS-conditional manifests handling platform-specific tooling (Chocolatey vs native packages, PowerShell vs Bash exec providers) while sharing the same Git workflow and secrets store.
* **Zero-trust network access** - SSH, the Puppet master's management interface, and Windows remote management (WinRM) are reachable only through a WireGuard VPN tunnel; there is no direct public exposure of administrative services. WinRM firewall rules are Puppet-managed and explicitly scoped to the VPN subnet, not left on default "any source" rules.
* **Observability** - a Zabbix stack monitors the environment, including:
  * Low-Level Discovery (LLD) for automatic container inventory/monitoring
  * Custom checks for Puppet Server JVM health (heap, JRuby pool, threads, GC)
  * Custom checks for deployment freshness (r10k last-successful-deploy age)

## Why I built it this way

Most "home lab" projects stop at "I installed X." This one is meant to practice the operational discipline around configuration management - branching strategy, secret hygiene, least-privilege network access, safe rollout of access-affecting changes, and monitoring - not just getting Puppet to run once.

A concrete example: extending the lab to manage a Windows node's remote-access surface (WinRM) meant every change had to be validated with `--noop` dry-runs and manually verified over the actual access path before being applied for real, since a bad firewall rule could cut off the only route into that host. That discipline - test before you trust the automation with your own access - is the point of the lab, not an afterthought.

## Security Notes

This is a live, private lab, not a disposable demo environment. To keep it that way while still being transparent about the work:

* Real hostnames, domains, and IP ranges are not published anywhere in this repo or the linked portfolio site.
* The actual Puppet control repo (manifests, hiera data, module code) is private and is not mirrored here.
* No credentials, tokens, or key material appear in this repo - encrypted secrets live only in the private control repo, scoped to their environment.

Want to see it live? I'm happy to do a screen-share walkthrough, or issue a time-limited, scoped VPN/dashboard-only guest credential for review - just reach out.

## Stack

Puppet - r10k - hiera-eyaml - WireGuard - Zabbix - Docker - Chocolatey
