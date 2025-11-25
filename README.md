# Crypto Broker Custom Buildpack

This is a custom Cloud Foundry supply buildpack for deploying the Crypto Broker application (server + client) as a sidecar pattern.

## Overview

This buildpack:
1. Builds the Go server component (`crypto-broker-server`)
2. Works with the Node.js buildpack to build the client component (`crypto-broker-cli-js`)
3. Sets up environment variables for runtime configuration
4. Enables sidecar deployment pattern

## Usage

### Option 1: Using the buildpack from a Git repository

If you push this buildpack to a Git repository (e.g., GitHub), you can reference it directly without admin privileges:

```bash
cf push -b https://github.com/YOUR_ORG/buildpack-crypto-broker.git -b nodejs_buildpack
```

Or use the `manifest.yml` with buildpack URLs:

```yaml
buildpacks:
  - https://github.com/YOUR_ORG/buildpack-crypto-broker.git
  - nodejs_buildpack
```

### Option 2: Using the buildpack locally

You can also reference the buildpack directory locally:

```bash
cd /path/to/workspace
cf push -b file://$(pwd)/buildpack-crypto-broker -b nodejs_buildpack
```

### Option 3: Package and upload as a Git buildpack

1. Create a separate repository for this buildpack
2. Copy the `buildpack-crypto-broker` directory contents to the repository root
3. Commit and push to your Git hosting service
4. Reference it in your deployment

## Buildpack Structure

```
buildpack-crypto-broker/
├── bin/
│   ├── detect    # Detects if this buildpack should be used
│   ├── supply    # Builds the Go server (supply phase)
│   ├── finalize  # Final build steps
│   └── release   # Defines process types
└── README.md     # This file
```

## Environment Variables

Set these in your `manifest.yml` or via `cf set-env`:

- `CRYPTO_BROKER_PROFILES_DIR`: Path to profiles directory (default: `/home/vcap/app/crypto-broker-server/example-profiles`)
- `CRYPTO_BROKER_LOG_LEVEL`: Logging level (default: `info`)
- `CRYPTO_BROKER_LOG_FORMAT`: Log format (default: `json`)
- `CRYPTO_BROKER_LOG_OUTPUT`: Log output destination (default: `stdout`)

## Multi-Buildpack Order

This buildpack must be used as a supply buildpack with the Node.js buildpack as the final buildpack:

1. **buildpack-crypto-broker** (supply) - Builds Go server
2. **nodejs_buildpack** (final) - Builds Node.js client

## Making Scripts Executable

Before deploying, ensure all scripts are executable:

```bash
chmod +x buildpack-crypto-broker/bin/*
```
