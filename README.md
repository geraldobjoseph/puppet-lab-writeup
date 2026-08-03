# Self-Managed Puppet Infrastructure Lab

A GitOps-driven Puppet environment I built and operate as a personal home lab —
designed to mirror how production configuration-management systems are actually run.

This repo documents the architecture and decisions. The working control repo,
hostnames, IP ranges, and credentials are intentionally kept private (see
[Security Notes](#security-notes) below).

## Overview

The lab consists of a Puppet master managing agent nodes over a hardened,
VPN-gated network, with a full GitOps deployment pipeline and monitoring stack.

Git Repo (dev/production branches)
|
v
GitHub Actions (lint -> validate -> deploy)
|
v (joins as ephemeral WireGuard peer)
Puppet Master (r10k + puppetserver, hiera-eyaml)
|
v (WireGuard tunnel)
Agent Node(s)
|
v
Zabbix Monitoring (LLD, custom checks, dashboards)


## What it does

- **GitOps workflow** - changes are made on a `dev` branch, reviewed via PR, and
  merged to `production`. Merges to `production` trigger an automated deploy.
- **CI/CD** - GitHub Actions lints and validates Puppet code on every PR. On
  merge, the deploy job joins the lab's WireGuard mesh as a short-lived,
  scoped peer, deploys via r10k, and then leaves the mesh - no standing
  credentials or always-on access from CI to the lab.
- **Secrets management** - all sensitive data (keys, credentials) is encrypted
  at rest using hiera-eyaml. Nothing sensitive is ever committed in plaintext.
- **Zero-trust network access** - SSH and the Puppet master's management
  interface are reachable only through a WireGuard VPN tunnel; there is no
  direct public exposure of administrative services.
- **Observability** - a Zabbix stack monitors the environment, including:
  - Low-Level Discovery (LLD) for automatic container inventory/monitoring
  - Custom checks for Puppet Server JVM health (heap, JRuby pool, threads, GC)
  - Custom checks for deployment freshness (r10k last-successful-deploy age)

## Why I built it this way

Most "home lab" projects stop at "I installed X." This one is meant to
practice the operational discipline around configuration management -
branching strategy, secret hygiene, least-privilege network access, and
monitoring - not just getting Puppet to run once.

## Security Notes

This is a live, private lab, not a disposable demo environment. To keep it
that way while still being transparent about the work:

- Real hostnames, domains, and IP ranges are not published anywhere in this
  repo or the linked portfolio site.
- The actual Puppet control repo (manifests, hiera data, module code) is
  private and is not mirrored here.
- No credentials, tokens, or key material appear in this repo - encrypted
  secrets live only in the private control repo, scoped to their environment.

**Want to see it live?** I'm happy to do a screen-share walkthrough, or issue
a time-limited, scoped VPN/dashboard-only guest credential for review - just
reach out.

## Stack

Puppet - r10k - hiera-eyaml - GitHub Actions - WireGuard - Zabbix - Docker
