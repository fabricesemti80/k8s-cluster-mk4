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

## VPN gateway

This will allow us to connect selected namespaces  or deployments to VPN.

For this you will need the files in **cluster/apps/vpn-gateway** folder, and also a few pre-requisites:

* I use NordVPN, so you should have an account with it (other VPN providers might need different setup)

* You should have a NordVPN config from here https://nordvpn.com/ovpn/

* You should prepare the following files:

    * **cluster/base/cluster-secrets.sops.yaml**

        * SECRET_VPN_COUNTRY: <this is the two character identifier for your VPN exit country>

    * **/home/fabrice/k8s-cluster-mk4/cluster/base/cluster-settings.yaml**

        * K8S_CLUSTER_CIDR: "10.42.0.0/16"

        * K8S_SERVICE_CIDR: "10.43.0.0/16"

        * NETWORK_MANAGEMENT_CIDR: "192.168.0.0/16" # this is in fact your LAN

        * VPN_GATEWAY_PORT: "1194"

        * VPN_PORT_QB: "56059" # only if you use qBittorrent

* Also create the file **cluster/apps/vpn-gateway/secret.sops.yaml** and fill it with your VPN config

## qBittorrent-vpn

This will allow us to download torrents via the vpn gateway.

Before doing this, we need to update the files:

* **cluster/base/cluster-secrets.sops.yaml**

    <span style="color:red">Important: these are just for the metrics! (You will need to log in first with **admin/adminadmin** first via the UI, and update the defaults to this! </span> (Will look into this...)

    * SECRET_QB_USERNAME

    * SECRET_QB_PASSWORD

* **/home/fabrice/k8s-cluster-mk4/cluster/base/cluster-settings.yaml**

    * NAS_SERVER

    * NAS_PATH_MEDIA # this is a path to an NFS share -  **a differnt one to the one used for StorageClass!** - where we keep our media data

We will also need to adjust the files in **/home/fabrice/k8s-cluster-mk4/cluster/apps/media/qbittorrent-vpn**

* update the storage class name to your prefered one in **cluster/apps/media/qbittorrent-vpn/config-pvc.yaml**

<span style="color:orange">Important2: at the moment QBittorrent redirection from Hajimari is not working, so you will need to reload the page, if you opened it from Hajimari. </span>

After the deployment is up:

* log in and change the web ui username and password

* adjust the default download path from /config/downloads/ to /media/downloads/

* connect to the pod and verify your contry

```sh
kah@qbittorrent-vpn-fd84b9b98-vldd9:/app$ curl ipinfo.io
{
  "ip": "217.138.xxx.xxx",
  "city": "Dublin",
  "region": "Leinster",
  "country": "IE",
  "loc": "53.3331,-6.2489",
  "org": "AS9009 M247 Ltd",
  "postal": "D02",
  "timezone": "Europe/Dublin",
  "readme": "https://ipinfo.io/missingauth"
}kah@qbittorrent-vpn-fd84b9b98-vldd9:/app$
```

*(fyi I am NOT in ireland, but my VPN exit is for NordVPN)*

## Sonarr

This will deal with our series management.

Sonarr is relatively straightforward, settings should be contained in folder **cluster/apps/media/sonarr**

## Plex

TBA

Update files:

* **cluster/base/cluster-settings.yaml**

    * SVC_PLEX_ADDRESS

* **cluster/base/cluster-secrets.sops.yaml**

    * SECRET_PLEX_CLAIM_TOKEN # from https://www.plex.tv/claim/

## Overseer

TBA
