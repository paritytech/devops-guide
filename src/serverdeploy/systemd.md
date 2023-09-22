# Systemd

## Overview

Systemd is a common way to manage services on Linux hosts. It can ensure the process is enabled and running, allows you to set a policy for restarts and also set the user running the host, limit the memory usage etc...

It can also use an environment variable file which can help abstract the variables into its own file per server.

## Simple Example

A simple example running a local dev chain as Alice, assuming the username is called polkadot would look like:

```yaml
[Unit]
Description=Polkadot Validator

[Service]
User=polkadot
ExecStart=/home/polkadot/polkadot  --dev --alice
Restart=always
RestartSec=90

[Install]
WantedBy=multi-user.target
```

This file should be placed in /etc/systemd/system/polkadot.service and then enabled with `systemctl enable polkadot` then `systemctl start polkadot` to start the service.

## Using Environment Variable Files

If we want to remove some options from the systemd file itself (e.g. `--dev --alice`) and put them in an environment variable file then the systemd service would now look like:

```yaml
[Unit]
Description=Polkadot Validator

[Service]
User=polkadot
EnvironmentFile=/etc/default/polkadot
ExecStart=/home/polkadot/polkadot  $START_OPTIONS
Restart=always
RestartSec=90

[Install]
WantedBy=multi-user.target
```

Then you would need to create a file in /etc/default/polkadot which looks like:

```bash
START_OPTIONS="--dev --alice"
```

You can do this with multiple variables to abstract the configuration from the systemd file running on your hosts.
