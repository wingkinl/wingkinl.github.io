---
title: "Setup Portainer in Synology with a web portal and custom domain"
date: 2023-04-28T15:49:38+08:00
tags: ['synology', 'docker', 'portainer']
categories: ["life"]
draft: false
---

I know there are already a lot of posts online that describe how to setup a Portainer on a Synology DSM, but I am writing this for my own reference.

## Create the docker image

SSH to the Synology, then run the following commands:

```bash
sudo -i

mkdir -p /volume1/docker/portainer

docker create --name=portainer --privileged --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /volume1/docker/portainer:/data portainer/portainer-ce
```



## Enable web portal

On Docker app, find the portainer container, then check **Enable web protal via Web Station**

Add port **9443** for HTTPS, then hit **Save**.

DSM will prompt to configure the web portal settings in Web Station.

![Enable web portal](web_portal.png)

Enter hostname for the web portal, then hit **Create**.

![Custom domain](custom_web_portal_domain.png)

Finally, start the container on Docker app and it's good to go.
