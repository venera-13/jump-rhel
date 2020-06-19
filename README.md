# JUMP is a Universal Maintenance Pod

**This version is based on Red Hat Enterprise Linux!**   
*For the version built on CentOS, see: https://github.com/venera-13/jump*

JUMP is a container originally designed as a tool for maintenance on ''edge'' systems running Podman. Starting a JUMP maintenance container can give access for troubleshooting or maintenance using the remote access tools most engineers are comfortable with. Using multiple protocols provides flexibility and allows access from numerous clients from a great number of operating systems.

Jump provides the following protocols for accessing the desktop:

- RDP
- VNC
- HTTPS (works through web proxies)

The JUMP container enables dynamic resizing of desktop sessions on all of the above protocols. No more scroll bars on your remote session!

## Note on Red Hat Universal Base Image

This container is built using the Red Hat Universal Base Image (UBI) (https://developers.redhat.com/products/rhel/ubi/). Red Hat provides support on containers built using their base images, which can be valuable in enterprise environments. A downside of using the UBI is that certain packages can only be accessed with an active Red Hat subscription. For this reason this container image can not be redistributed through, for example, Docker Hub.

There are a few extra steps for building UBI images using packages only available through subscription:

1. Set up a Red Hat 8 system with active subscription
2. Install Podman
3. Build the image

Podman passes on the subscription details of the host on which the container image is built to the microdnf package manager in the UBI. This allows access to all packages. Developers can create a free Red Hat developers account which provides a subscription, access to Red Hat ISO images and access to registry.redhat.io. For more information: https://developers.redhat.com/

Quick guide to set up a Red Hat Enterprise Linux 8 build host:

```bash
# Register your system and attach a subscription
subscription-manager register
subscription-manager refresh
subscription-manager attach --auto

# Enable codeready-builder repository which is needed by EPEL
ARCH=$( /bin/arch )
subscription-manager repos --enable "codeready-builder-for-rhel-8-${ARCH}-rpms"

# Install EPEL repository
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# Install podman to build and run containers
dnf install podman

# Login to the Red Hat registry to pull UBI images
podman login registry.redhat.io
```

## Encryption

The HTTPS and RDP protocols enforce encryption by default. TigerVNC offers anonymous TLS encryption but does not mind clients connecting without using it. To enforce the use of TLS in VNC the variable "VNCTLS" in the Containerfile can be set to "true". Please note this breaks the noVNC HTTPS frontend and some older VNC clients.

## Used components

- Red Hat Enterprise Linux 8 Universal Base Image
- TigerVNC-server
- noVNC HTML5 VNC-frontend
- XRDP Terminal Server for X11
- Chromium browser
- Openbox window manager

## Build instructions

Clone the Git repository and add your own values to the "ARG" variables in the Containerfile.

| ARG        | Description                              |
|------------|------------------------------------------|
| USER       | Username of desktop user                 |
| PASS       | Password of desktop user                 |
| TZ         | Timezone the container lives in          |
| KEYSUBJECT | Information about the SSL certificate    |
| VNCTLS     | Enforce TLS encryption on VNC server     |
| BROWSER    | Select your browser: firefox/chromium    |
| OPTPKGS    | Select optional packages to install      |


Building the container with podman

```bash
git clone https://github.com/venera-13/jump-rhel.git
cd jump-rhel
vim Containerfile
podman build -f ./Containerfile -t localhost/jump-rhel:latest
```

## Using the container

Start the container with the below command. Parameters are explained in the table.

```bash
podman run -d --rm --shm-size=1g --name jump  -p 3389:3389 -p 5901:5901 -p 8080:8080 localhost/jump-rhel:latest
podman stop jump
```

| Parameter     | Omschrijving                                                       |
|---------------|--------------------------------------------------------------------|
| -d            | Run container in background non interactively                      |
| --rm          | Delete container after you're done with it                         |
| --shm-size=1g | Increase allowed shared memory. Keeps modern browsers from crashing|
| --name jump   | Name the container so it is easy to recognise                      |
| -p 3389:3389  | Expose RDP port                                                    |
| -p 5901:5901  | Expose VNC port                                                    |
| -p 8080:8080  | Expose port for noVNC HTML5 VNC frontend                           |

After starting, the container is ready to accept connections from RDP, VNC or Web clients.
