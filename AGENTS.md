# Agents Guide

## Project Overview

systemcheck is a system health check tool for Kicksecure and Whonix.
It runs a series of diagnostic functions and reports results via CLI and GUI (msgcollector).

## Architecture

- Shell scripts in `usr/libexec/systemcheck/` are sourced by the main `systemcheck` script.
- `preparation.bsh` is sourced first and provides shared helpers.
- `canary-download` is a standalone Python script that runs as user `canary` under AppArmor confinement.
- `log-checker` is a standalone bash script invoked via `leaprun` (privilege escalation wrapper).
- Configuration is sourced from `/etc/systemcheck.d/*.conf` and `/usr/local/etc/systemcheck.d/*.conf`.

## Security review guidelines

See `agents/security.md`.
