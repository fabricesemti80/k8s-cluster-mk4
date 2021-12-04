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

Take a look on the structure under 'cluster/apps/networking/external-dns'
