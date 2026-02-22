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

### Build-Time Variables

These environment variables **must be set** before deployment (in `manifest.yml` or as system environment variables):

- **`BP_NODE_APP_DIR`** (required): Directory name of your Node.js application
  - Allows you to use this buildpack with different Node.js applications
  - Example: `BP_NODE_APP_DIR=crypto-broker-cli-js` or `BP_NODE_APP_DIR=my-custom-app`
- **`BP_NODE_APP_ENTRY`** (required): Entry point file path relative to the Node.js app directory
  - Specifies the entry point for your Node.js application
  - Example: `BP_NODE_APP_ENTRY=dist/cli.js` or `BP_NODE_APP_ENTRY=src/index.js`

### Runtime Variables

Set these in your `manifest.yml` or via `cf set-env`:

- `CRYPTO_BROKER_PROFILES_DIR`: Path to profiles directory (default: `/home/vcap/app/crypto-broker-server/example-profiles`)
- `CRYPTO_BROKER_LOG_LEVEL`: Logging level (default: `info`)
- `CRYPTO_BROKER_LOG_FORMAT`: Log format (default: `json`)
- `CRYPTO_BROKER_LOG_OUTPUT`: Log output destination (default: `stdout`)

## Using with Different Node.js Applications

This buildpack is now parametrizable! You can use it with any Node.js application that integrates the crypto-broker-client library.

### For `crypto-broker-cli-js`:

```yaml
buildpacks:
  - https://github.com/YOUR_ORG/buildpack-crypto-broker.git
  - nodejs_buildpack
env:
  BP_NODE_APP_DIR: crypto-broker-cli-js
  BP_NODE_APP_ENTRY: dist/cli.js
  CRYPTO_BROKER_PROFILES_DIR: /home/vcap/app/crypto-broker-server/example-profiles
```

### For a custom application (e.g., `my-node-app`):

```yaml
buildpacks:
  - https://github.com/YOUR_ORG/buildpack-crypto-broker.git
  - nodejs_buildpack
env:
  BP_NODE_APP_DIR: my-node-app
  BP_NODE_APP_ENTRY: src/index.js
  CRYPTO_BROKER_PROFILES_DIR: /home/vcap/app/crypto-broker-server/example-profiles
```

Your application structure should look like:

```
my-app/
├── crypto-broker-server/    # Go server (required)
├── my-node-app/              # Your Node.js application
│   ├── package.json          # Your app's dependencies
│   └── src/
│       └── index.js          # Entry point (or dist/cli.js, etc.)
├── package.json              # Root package.json (required - see below)
└── manifest.yml
```

**Important:** The root `package.json` is required for the buildpack to work correctly. It must contain a `postinstall` script that installs dependencies in your Node.js application directory:

```json
{
  "name": "crypto-broker-app",
  "version": "1.0.0",
  "private": true,
  "engines": {
    "node": ">=18.x"
  },
  "scripts": {
    "postinstall": "cd ${BP_NODE_APP_DIR} && npm install --production"
  }
}
```

This script ensures that the Node.js buildpack installs dependencies in the correct directory specified by `BP_NODE_APP_DIR`.

## Multi-Buildpack Order

This buildpack must be used as a supply buildpack with the Node.js buildpack as the final buildpack:

1. **buildpack-crypto-broker** (supply) - Builds Go server
2. **nodejs_buildpack** (final) - Builds Node.js client

## Making Scripts Executable

Before deploying, ensure all scripts are executable:

```bash
chmod +x buildpack-crypto-broker/bin/*
```
