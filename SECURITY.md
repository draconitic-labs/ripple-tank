# Security policy

## Supported versions

Security fixes are made on the latest release line.

## Reporting a vulnerability

Do not put exploit details in a public issue. Email [support@draconitic.com](mailto:support@draconitic.com) with the affected version, operating system, a minimal reproduction and the expected impact. You should receive an acknowledgment within seven days; timing for a fix depends on severity and reproducibility.

## Security boundary

Ripple Tank runs locally. The engine has no network access, and experiment files are treated as untrusted input: malformed state is rejected and outputs refuse to overwrite existing files. The browser version does not upload experiments.
