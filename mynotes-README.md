# Combined notes


## I.  WSL preparation

Update the system

```sh
sudo apt-get update -y && sudo apt-get upgrade -y
```

# Install Git

```sh
sudo apt-get install git -y && git --version
```

# Install Homebrew

```sh

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Run these two commands in your terminal to add Homebrew to your PATH:
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/fabrice/.profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# Install Homebrew's dependencies if you have sudo access:
    sudo apt-get install build-essential -y
    #For more information, see:
    #https://docs.brew.sh/Homebrew-on-Linux

# We recommend that you install GCC:
    brew install gcc

# Run brew help to get started

# Further documentation:
    # https://docs.brew.sh

```

*[source](https://brew.sh/)*


## SSH

Copy the SSH key(s) to the .ssh folder

```sh

mkdir .ssh

# In this case I am copying from the Windows host machine

┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ cp /mnt/c/Users/emilf/.ssh/id_ed25519 .ssh/id_ed25519

┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ cp /mnt/c/Users/emilf/.ssh/id_ed25519.pub .ssh/id_ed25519.pub


┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ ls -la .ssh/
total 16
drwxr-xr-x 2 fabrice fabrice 4096 Dec  3 14:44 .
drwxr-xr-x 6 fabrice fabrice 4096 Dec  3 14:42 ..
-rwxr-xr-x 1 fabrice fabrice  419 Dec  3 14:44 id_ed25519
-rwxr-xr-x 1 fabrice fabrice  106 Dec  3 14:44 id_ed25519.pub

```

Fix file permissions

```sh
┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ chmod 600 /home/fabrice/.ssh/id_ed25519

┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ chmod 600 /home/fabrice/.ssh/id_ed25519.pub 
```

Add ssh key

```sh
┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ eval "$(ssh-agent -s)"
Agent pid 29183

┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ ssh-add ~/.ssh/id_ed25519
Identity added: /home/fabrice/.ssh/id_ed25519 (fabrice@fabricesemti.com)
```

Ensure SSH auto starts

```sh
#? SSH autostart - https://unix.stackexchange.com/questions/90853/how-can-i-run-ssh-add-automatically-without-a-password-prompt
if [ -z "$SSH_AUTH_SOCK" ] ; then
    eval `ssh-agent -s`
    ssh-add
fi
```

Create an ssh config for Git

```sh
host github.com
HostName github.com
IdentityFile ~/.ssh/id_ed25519
User git

host k8s-0
HostName 192.168.1.10
IdentityFile ~/.ssh/id_ed25519
User ubuntu

host k8s-1
HostName 192.168.1.20
IdentityFile ~/.ssh/id_ed25519
User ubuntu

host k8s-2
HostName 192.168.1.21
IdentityFile ~/.ssh/id_ed25519
User ubuntu

host k8s-3
HostName 192.168.1.22
IdentityFile ~/.ssh/id_ed25519
User ubuntu
```

Test connections 

```sh
┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ ssh -T git@github.com
The authenticity of host 'github.com (140.82.121.3)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi fabricesemti80! You've successfully authenticated, but GitHub does not provide shell access.

┌──(fabrice㉿DESKTOP-FS)-[~]
└─$ ssh ubuntu@192.168.1.10
The authenticity of host '192.168.1.10 (192.168.1.10)' can't be established.
ED25519 key fingerprint is SHA256:FblB/wrvJ6882hVdGN7yDWH2stPwOnItqmJd7ffKD/o.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.10' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri  3 Dec 15:01:32 UTC 2021

  System load:           0.98
  Usage of /:            21.5% of 113.37GB
  Memory usage:          54%
  Swap usage:            0%
  Temperature:           52.0 C
  Processes:             284
  Users logged in:       0
  IPv4 address for eth0: 192.168.1.10
  IPv4 address for eth0: 192.168.1.254
  IPv6 address for eth0: fdaa:bbcc:ddee:0:2e4d:54ff:fe65:3dec

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

0 updates can be applied immediately.


Last login: Fri Dec  3 12:46:58 2021 from 192.168.2.51
ubuntu@k8s-0:~$ exit
logout
Connection to 192.168.1.10 closed.

...

```

## Install brew packages

I have a list of binaries I like to use, install these or some of them, if you like

```sh
xargs brew install < /home/fabrice/k8s-cluster-mk4/provision/brew-packages.txt
```
## II. Prepare the nodes

### Networking


Ensure that the k8s servers have the same named network interfaces (this is important for kubevip).

Elevate your session

```sh
sudo su
```

Create netplan

```yaml
---
#! /bin/bash
var=/etc/netplan/01-network-manager-all.yaml
cat << EOF > $var
# Let NetworkManager manage all devices on this system
# ref: https://askubuntu.com/questions/1317036/how-to-rename-a-network-interface-in-20-04
# update madaddress (and add further interfaces, if you have any) accordingly
# sudo nano /etc/netplan/01-network-manager-all.yaml 
  network:
    version: 2
    ethernets:
        eth0:            
            addresses:
            #TODO: upate this for the selected IP of the host
            - 192.168.1.22/16 
            gateway4: 192.168.1.1
            nameservers:
                addresses:
                - 192.168.1.1
                - 208.67.222.222
                - 208.67.220.220
                search: []
            match:
                #TODO: upate this for the active NIC of the host
                macaddress: b0:83:fe:ba:ac:d8
            set-name: eth0
        # eth1:
            # dhcp4: true
            # match:
                # macaddress: 2c:4d:54:65:3d:eb
            # set-name: eth1
EOF
```
Apply this config and reboot

```sh
netplan apply && reboot
```

After the system restarted, you should be able to log in with the new IP remotely.

## Clone the repository and adjust

[repo - as of 2021-12-03](https://github.com/fabricesemti80/k8s-cluster-mk4)

This is the repository where the K8S code lives

```sh
─(fabrice㉿DESKTOP-FS)-[~/k8s-cluster-mk4]
└─$ git clone git@github.com:fabricesemti80/k8s-cluster-mk4.git
```

### Prepare pre-commit

Refer to the README.md  for this. (Adjust file /home/fabrice/k8s-cluster-mk4/.pre-commit-config.yaml if needed)

### Follow the readme

Follow the steps described there, until you complete the "Preparing Ubuntu with Ansible section"

## K3S install

Before proceeding, edit this file: 
provision/ansible/inventory/group_vars/kubernetes/k3s.yml

Set this to true, if you have **less than 3 master nodes**

```yaml
# (bool) Allow the use of unsupported configurations in k3s
k3s_use_unsupported_config: true
```
[reference](https://githubmemory.com/repo/k8s-at-home/template-cluster-k3s/issues/115)

Verify the installation was succesfull

```sh
fabrice@DESKTOP-FS:~/k8s-cluster-mk4$ kubectl --kubeconfig=./provision/kubeconfig get nodes
NAME    STATUS     ROLES                       AGE    VERSION
k8s-0   Ready      control-plane,etcd,master   112s   v1.22.3+k3s1
k8s-1   NotReady   <none>                      16s    v1.22.3+k3s1
k8s-2   NotReady   <none>                      16s    v1.22.3+k3s1
k8s-3   NotReady   <none>                      16s    v1.22.3+k3s1
fabrice@DESKTOP-FS:~/k8s-cluster-mk4$
```

Copy the kubeconfig

```sh
fabrice@DESKTOP-FS:~/k8s-cluster-mk4$ cp ./provision/kubeconfig ~/.kube/config
```

Some additional considerations to the .bashrc (for zshell some of these might differ)

```sh
#& HOMEBREW
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/fabrice/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

#& SSH
if [ -z "$SSH_AUTH_SOCK" ] ; then 
    eval `ssh-agent -s` 
    ssh-add 
fi

# #####################################################################################&
# ###&  The --clear option make sure Intruder cannot use your existing SSH-Agents keys 
# ###& i.e. Only allow cron jobs to use password less login 
# #####################################################################################&
# /usr/bin/keychain --clear $HOME/.ssh/id_ed25519
# source $HOME/.keychain/$HOSTNAME-sh

# ##& https://stackoverflow.com/questions/63793836/unable-to-commit-to-git-with-the-gpg-key-error
# GPG_TTY=$(tty)
# export GPG_TTY

##& https://github.com/ahmetb/kubectl-aliases
[ -f ~/.kubectl_aliases ] && source \
   <(cat ~/.kubectl_aliases | sed -r 's/(kubectl.*) --watch/watch \1/g')
function kubectl() { echo "+ kubectl $@">&2; command kubectl $@; }   

#& https://fluxcd.io/docs/cmd/flux_completion_bash/
command -v flux >/dev/null && . <(flux completion bash)

#& https://neate.dev/technology/2019/11/14/changing-the-default-editor-of-kubectl-edit.html
export KUBE_EDITOR=code

alias cls=clear

#& https://github.com/hidetatz/kubecolor
alias kubectl="kubecolor"

#& backup this profile
yes | cp ~/.bashrc /home/fabrice/k8s-home-cluster-mk3/hack/.bashrc_backup

#& https://github.com/kubernetes/kubernetes/issues/17512
alias util='kubectl get nodes --no-headers | awk '\''{print $1}'\'' | xargs -I {} sh -c '\''echo {} ; kubectl describe node {} | grep Allocated -A 5 | grep -ve Event -ve Allocated -ve percent -ve -- ; echo '\'''
# # Get CPU request total (we x20 because because each m3.large has 2 vcpus (2000m) )
# alias cpualloc='util | grep % | awk '\''{print $1}'\'' | awk '\''{ sum += $1 } END { if (NR > 0) { print sum/(NR*20), "%\n" } }'\'''
# # Get mem request total (we x75 because because each m3.large has 7.5G ram )
# alias memalloc='util | grep % | awk '\''{print $5}'\'' | awk '\''{ sum += $1 } END { if (NR > 0) { print sum/(NR*75), "%\n" } }'\'''

#&
# k = kubectl  https://kubernetes.io/docs/reference/kubectl/cheatsheet/
alias k=kubectl
complete -F __start_kubectl k
```

## Flux install

Follow the other readme.md

