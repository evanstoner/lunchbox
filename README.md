# lunchbox

_Setup for a compact OpenShift demo node aka lunchbox._

## One time router setup

1. Start up your GL-AR750S and plug in or connect to its wifi
2. Add the following to your `~/.ssh/config` to allow OpenWRT's only (insecure) crypto:

  ```
  Host 192.168.8.1
      PubkeyAcceptedAlgorithms +ssh-rsa
      HostkeyAlgorithms +ssh-rsa
  ```

3. `ssh root@192.168.8.1`
4. Effectively wildcard the entire `demo.local` domain to 192.168.8.10, which will
   be the SNO cluster's IP (we'll configure that later):

  ```
  uci add_list dhcp.@dnsmasq[0].address="/apps.sno.demo.local/192.168.8.10"
  uci add_list dhcp.@dnsmasq[0].address="/api.sno.demo.local/192.168.8.10"
  uci commit dhcp
  /etc/init.d/dnsmasq restart
  exit
  ```

## Prereqs

- The NVMe disk on the node must be empty. If not, LVM storage will not provision
    itself. If you can't format this disk before the install, you can do so after
    and let the LVM operator reconcile itself.

## Get started

Determine the MAC address of the network interface you'll use for the node. If
you don't know, just plug it in to the router, boot it up, and check the clients
list in the router's web GUI.

**Create a cluster using Assisted Installer at https://console.redhat.com**

1. Cluster details:
  1. Cluster name: `sno`
  1. Base domain: `demo.local`
  1. Install SNO
  1. Hosts' network configuration: Static IP
1. Network-wide configurations:
  1. DNS: `192.168.8.1`
  1. Machine network: `192.168.8.0` / `24`
  1. Default gateway: `192.168.8.1`
1. Host-specific configurations:
  1. MAC address: enter the MAC you determine for the node
  1. IP address: `192.168.8.10`
1. Operators: install OpenShift Virt and LVM Storage
1. Host discovery: Add host
  1. Provisioning type: full image file
  1. SSH public key: add your `~/.ssh/id_rsa.pub` here
  1. Generate Discovery ISO
1. Burn ISO to a flash drive, [Belena Etcher](https://www.balena.io/etcher) is easy

**Install the cluster**

1. Insert the flash drive and boot from it _one time_ (you can get a virtual console
    from the BMC web GUI, F11 launches the boot menu)
1. After a few minutes the node shows up in the Red Hat Console, click Next, Next,
    Next, and finally Install cluster
1. Wait until the installer is 100% complete, then copy the `kubeadmin` password
    displayed. Also download the `kubeconfig` file.
1. Navigate to https://console-openshift-console.apps.sno.demo.local/, accept
    the certificates, and log in with `kubeadmin` and the password you just copied.

**Smoke test**

1. Check that all operators are Succeeded
1. Check that the LVM Storage operator's `LVMCluster` is Ready.

**Bootstrap the cluster**

1. Apply bootstrap/10-gitops.yaml (either from the command line or the **+** in
    the web console). Wait for Upgrade status to show Up to date.
1. Apply bootstrap/20-application.yaml

**Currently manual VM stuff**

Create Service and Route for cockpit, start cockpit.socket, subscription-manager register,
dnf install -y container-tools git

Download and unzip https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz, move to /usr/local/bin

Clone https://gitlab.com/evanstoner/ocp-intro.git

Follow DEMO.md and then ocp-intro README.md
