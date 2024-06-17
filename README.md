# Tailscale in codespaces
- simply following https://tailscale.com/kb/1160/github-codespaces
- needed to run `sudo tailscale up --accept-routes`

# Prereqs
- add windows local device to tailnet
- add codespace to tailnet using [devcontainer.json](./.devcontainer/devcontainer.json)

## Test windows to codespace
- on windows `ping <codespace-tailscale-ip>`

## Test codespace to windows
- on windows, setup server `python -m http.server 8000`
- on codespace `curl -Is http://<windows-tailscale-ip>:8000`