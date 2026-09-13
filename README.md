Self-Managed Puppet Infrastructure Lab

A GitOps-driven Puppet environment I built and operate as a personal home lab — designed to mirror how production configuration-management systems are actually run.

This repo documents the architecture and decisions. The working control repo, hostnames, IP ranges, and credentials are intentionally kept private (see Security Notes below).

Overview

The lab consists of a Puppet master managing both Linux and Windows agent nodes over a hardened, VPN-gated network, with a full GitOps deployment pipeline and monitoring stack.
