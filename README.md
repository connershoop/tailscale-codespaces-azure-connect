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

## Test codespace to azure vm

- in azure create a virtual network and storage account.
- follow this guide to setup tailscale client on an azure vm: https://tailscale.com/kb/1142/cloud-azure-linux
    - make sure to properly allow IP fowarding on the VM by configuring your vm following this guide: https://tailscale.com/kb/1019/subnets?tab=linux#advertise-subnet-routes
    - run your azure vm as an exit node also with `sudo tailscale up --advertise-routes=10.0.0.0/24,168.63.129.16/32 --accept-dns=false` and `sudo tailscale set --advertise-exit-node`
- in codespaces or locally test your connection with: `sudo tailscale up --accept-routes` to setup your client and `ssh -i ./keys/vm-tailscale-sandbox-00_key.pem azureuser@vm-tailscale-sandbox-00` to connect to your new tailnet azure vm 
- run `sudo tailscale set --exit-node=100.88.190.102` to use the azure vm as an exit node