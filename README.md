# GCP development VM

## Register SSH public key with GCP

This registers the `mozilla.pub` public key with Google Cloud OS Login for the `carbon-cacao-475319-k4` project with an unlimited lifetime.

```bash
gcloud compute os-login ssh-keys add \
    --key-file=$HOME/.ssh/mozilla.pub \
    --project=carbon-cacao-475319-k4 \
    --ttl=0
```

You obviously need to make your own SSH key pair, and check what project name your VM is in.

## Configure SSH client for GCP IAP tunneling

Add something like this to your `~/.ssh/config` file:

```text
Host moz-fx-leggert
    Hostname moz-fx-leggert
    User lars
    IdentityFile ~/.ssh/mozilla
    ProxyCommand gcloud compute start-iap-tunnel %h 22 --listen-on-stdin --project=carbon-cacao-475319-k4 --zone=europe-north1-b
```

Adjust the host name, username, private key, project name and zone as necessary. You should then be able to log into the VM with:

```bash
ssh moz-fx-leggert
```

## Ansible configuration

Use the Ansible playbook in `ubuntu.yml` to configure the VM:

```bash
ansible-playbook -i inventory.yml ubuntu.yml
```

Refrain from changing the VM configuration directly on the VM; instead, make changes to the Ansible playbook and re-run it.
