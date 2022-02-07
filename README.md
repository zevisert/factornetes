# Factorio on Kubernetes

This repository contains deployment manifests to run a factorio server on kubernetes. Run `kubectl -f kubernetes` from within the top of the repository to deploy. These manifests are ready to deploy, no templating tools are used. The game server will run the `factoriotools/factorio` image. 

## Mods

This repo contains a mod downloader script to grab mods in an initContainer that runs before the server starts. The mods are pulled from mods.factorio.com, but doing so requires a secret in the same namespace called `factorio-auth` with `username` and `token` keys. 

Currently the mods to install are just listed as command-line arguments to this script;

Current mods:
- far-reach
- Flow Control
- bullet-trails
- informatron
- AutoDeconstruct
- even-distribution
- Bottleneck
- LogisticTrainNetwork
- Squeak Through

## Secrets

The following secrets are required (obviously replace the `null` fields with your bas64 values)

```yaml
# To download mods from mods.factorio.com
kind: Secret
apiVersion: v1
metadata:
  name: factorio-auth
  namespace: factorio
data:
  token: null
  username: null
---

# RCON Password
apiVersion: v1
kind: Secret
metadata:
  name: rcon-password
  namespace: factorio
data:
  rconpw: null

---

# Game config files
apiVersion: v1
kind: Secret
metadata:
  name: settings
  namespace: factorio
data:
  # These should be contents of the files,
  # Map and server settings are secrets are to protect seeds, etc
  map-gen-settings.json: null
  map-settings.json: null
  server-adminlist.json: null
  server-banlist.json: null
  server-settings.json: null

```

## Requirements

This project leverages `sealed-secrets` and `external-dns`, though both are optional. 

### Without `sealed-secrets`

You'll have to deploy secrets directly, I chose `sealed-secrets` so that I could make this repository public while still tracking the secrets. They're just encrypted.

### Without `external-dns`

The game's UDP port is exposed via services of `type: NodePort` on 30000 since the default port of 34197 isn't available for `NodePort` binding, additionally, the cloud provider I'm using doesn't have UDP load balancing, so `type: LoadBalancer` isn't an option either.

The TCP `rcon` port is 30001.

## TODO

- [ ] Use a configmap to configure mods to install
- [ ] Removing a mod from the configmap should uninstall the mod
- [ ] Removing a mod should restart the server
- [ ] Find a way to host UDP on the game's default port without needing to use `HostPort`
- [ ] Enable the use of the player whitelist
- [ ] Make whitelist and banlist optional