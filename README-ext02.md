# Once the cluster is up

## FLUX alerts

We will configure Flux to send alerts to a discord channel. I will not cover setting up Discord or creating a channel, as that should be trivial.

Once we have a channel and a webhook, we can proceed.

Please take a look on the files and folders under "cluster/base/flux-system/notifications"

* this structure should be created

* you need to encrypt the secret file, as per the comment (modify the path!)

* also add the "notifications" folder into "cluster/base/flux-system/kustomization.yaml"


## Networking

### External DNS

We will create external DNS so that our ingresses are automatically updated on CloudFlare (you might need a different method, if you are using a different provider!)

Take a look on the structure under **cluster/apps/networking/external-dns**

* update the secret with your values

* encrypt the file

* also add the **external-dns** folder to the **cluster/apps/networking/kustomization.yaml** file

### Traefik forward auth / Traefik Middleware

Create the structure as per the folders.

* update the secret with your values

* encrypt the file

* also add the **middleware and **traefik-forward-auth** folders to the **cluster/apps/networking/traefik/kustomization.yaml** file

Once this is done, the ingresses should ask for Google authentication.


### Cloudflare DDNS

This should be also set up, so that the cluster always - even if the external IP of your router changes - is having up-to-date DNS record.

If this does not work, the alternative is to run Terraform as per the README.md

```sh
task terraform:apply:cloudflare
```

## Storage

In my case I have a Qnap NAS, that serves as my NFS provider, so I opted for a simple NFS storage class. (If you have other type of storages, you need to set up the storage class differently)

The neccesary files can be found under **cluster/apps/storage**.

* you will also need to include your NAS-s IP or hostname in this file: **cluster/base/cluster-settings.yaml**

* also make sure to update the paths on the NFS server in file **cluster/apps/storage/storageClass/provisioner-deploy.yml**

ref: https://levelup.gitconnected.com/how-to-use-nfs-in-kubernetes-cluster-storage-class-ed1179a83817

Note: you will need to replace the arm image, if you are not using Raspberry PI. https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
