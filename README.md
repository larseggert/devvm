# GCP development VM

## Make a GCP VM for development

Make a new GCP project for your dev VM at <https://console.cloud.google.com/projectcreate> and give it a name, e.g., `$DEVVM`. Note the generated project ID, e.g., `$PROJECT`.

Go to <https://console.cloud.google.com/compute> and create a new VM instance. I happen to use a `c2-standard-30` machine type; pick whatever works for you. Use a zone near where you'll mostly access the VM from for low latency access, I'll refer to it as `$ZONE` below.

The OS needs to be Ubuntu if you want to use the provided Ansible playbook, I used Ubuntu 25.10.

## Install `gcloud` CLI

Install the Google Cloud SDK (`gcloud` CLI) on your local machine by following the instructions at: <https://cloud.google.com/sdk/docs/install>. Authenticate with your Google account as described at: <https://cloud.google.com/docs/authentication/gcloud>.

## Register SSH public key in GCP project metadata

`$USER` below is your desired username on the VM. `$PUBLIC_KEY` is the contents of the SSH public key file you would like to use to log into the VM.

Run this command to register the SSH public key in the GCP project metadata:

```bash
gcloud compute project-info add-metadata --metadata="ssh-keys=$USER:$PUBLIC_KEY"
```

## Configure SSH client for GCP IAP tunneling

Add something like this to your `~/.ssh/config` file, with all variables replaced as defined above, and `$PRIVATE_KEY` as the filename of the private key corresponding to the public key you registered above.

```bash
Host $DEVVM
    User $USER
    IdentityFile ~/.ssh/$PRIVATE_KEY
    ProxyCommand gcloud compute start-iap-tunnel %h %p --listen-on-stdin --project=$PROJECT --zone=$ZONE
    ForwardX11 yes # If you want X11 forwarding
    ForwardAgent yes # If you want SSH agent forwarding
```

Adjust the host name, username, private key, project name and zone as necessary. You should then be able to log into the VM with:

```bash
ssh $DEVVM
```

## Ansible configuration

Use the Ansible playbook in `ubuntu.yml` to configure the VM:

```bash
ansible-playbook -i $DEVVM, ubuntu.yml
```

## Notes

The playbook configures the VM to log you out after 30 minutes of inactivity, and it will shut down the VM after at most another 30 minutes of inactivity. This is to save costs when you forget to turn off the VM. You can restart the VM with:

```bash
gcloud compute instances start $DEVVM
```

Refrain from changing the VM configuration directly on the VM; instead, make changes to the Ansible playbook and re-run it.
