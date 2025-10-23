# GCP development VM

## Make a GCP VM for development

Make a new GCP project for your dev VM at <https://console.cloud.google.com/projectcreate>. Note the generated project ID, e.g., `carbon-cacao-123456-xy`.

Go to <https://console.cloud.google.com/compute> and create a new VM instance. I happen to use a `c2-standard-30` machine type in the `europe-north1-b` zone. Pick whatever works for you; use a zone near where you'll mostly access the VM from for low latency access. The OS needs to be Ubuntu if you want to use the provided Ansible playbook, I used Ubuntu 25.10.

## Register SSH public key with GCP

This registers the `sshkey.pub` public key with Google Cloud OS Login for the `carbon-cacao-123456-xy` project with an unlimited lifetime.

```bash
gcloud compute os-login ssh-keys add \
    --key-file=$HOME/.ssh/sshkey.pub \
    --project=carbon-cacao-123456-xy \
    --ttl=0
```

You obviously need to make your own SSH key pair, and check what project name your VM is in.

## Configure SSH client for GCP IAP tunneling

Add something like this to your `~/.ssh/config` file:

```text
Host moz-fx-leggert
    Hostname moz-fx-leggert
    User lars
    IdentityFile ~/.ssh/sshkey
    ProxyCommand gcloud compute start-iap-tunnel %h 22 --listen-on-stdin --project=carbon-cacao-123456-xy --zone=europe-north1-b
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

The playbook configures the VM to log you out after 30 minutes of inactivity, and it will shut down the VM after at most another 30 minutes of inactivity. This is to save costs when you forget to turn off the VM.

Refrain from changing the VM configuration directly on the VM; instead, make changes to the Ansible playbook and re-run it.
