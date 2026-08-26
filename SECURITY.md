# Security Policy

## Lab-Only Scope

This repository documents a private, simulated cybersecurity lab. All scanning, firewall testing, endpoint telemetry generation, and threat-hunting activity were performed only against systems owned and controlled by the lab operator.

## Responsible Use

The material here is intended for defensive security learning, portfolio development, and systems administration practice. Do not use the commands, techniques, or procedures against networks or systems without explicit authorization.

## Sensitive Information Handling

The public repository is designed to contain sanitized evidence only. It should not include:

- Passwords or credential archives
- API keys, tokens, private keys, or session secrets
- Personal email addresses or phone numbers
- Public IP addresses tied to a real person or organization
- Windows SIDs, real usernames, or unnecessary local identifiers
- Unsanitized tool exports containing secrets

Private RFC1918 lab addresses such as `10.10.10.0/24` and `192.168.109.0/24` may appear because they are part of the documented lab topology.

## Reporting an Issue

If you notice exposed sensitive data or a documentation issue in this repository, please open a GitHub issue or contact the repository owner through GitHub.
