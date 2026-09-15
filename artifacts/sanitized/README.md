# Sanitized Public Artifacts

This directory contains evidence that is useful for independent review but safe for public release.

## Included

- concise certificate/public-key metadata;
- version and provider summaries;
- sanitized capture-derived summaries;
- hashes or identifiers that allow a public result to be mapped to privately retained raw evidence;
- non-sensitive experiment manifests.

## Excluded

The following must **not** be committed here:

- private keys;
- credentials, tokens or passwords;
- raw secrets or environment files;
- personally identifiable information;
- production or third-party traffic;
- raw PCAPs containing unnecessary addressing/payload information;
- full host inventories or sensitive internal network details.

## Evidence model

The public artifact is a reviewable derivative, not a substitute for raw research evidence. Raw artifacts are retained separately and integrity-linked with hashes in the experiment provenance record.

Each sanitized artifact should identify the phase/test ID it supports and should avoid claiming more than the preserved evidence establishes.
