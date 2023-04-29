---
title: "Auto Renew ZeroSSL in Synology"
date: 2023-04-30T00:02:02+08:00
categories: ['life']
tags: ['Synology', 'SSL', 'HTTPS', 'ZeroSSL', 'Certificate']
draft: false
---

## Register a ZeroSSL account and generate an EAB credential

Here's the link to create an EAB:

https://app.zerossl.com/develope

![ZeroSSL EAB](ZeroSSL_EAB.webp)

## Create a scheduled task to run a script that auto renew the certificate

### Prepare the script and folder

Create a folder **/volume1/docker/acme**

Put this script on the folder and name the script file as **my_update_ssl.sh**.

Note, I am using **5001** as **HTTPS** port for my DSM, you may change it or remove it if you use **HTTP** instead.

```sh
#! /bin/bash

export ACME_VOLUME=/volume1/docker/acme
export DOMAIN=YOUR_DOMAIN_NAME.com

echo ">>>>>>> Ken: ==== Begin of my SSL update script ===="

echo ">>>>>>> Ken: 1) Running acme.sh daemon..."

docker run --rm  -itd  \
  -v "${ACME_VOLUME}":/acme.sh  \
  --net=host \
  --name=acme.sh \
  neilpang/acme.sh daemon

echo ">>>>>>> Ken: 2) Registering ZeroSSL account"

docker exec acme.sh \
--register-account --server zerossl --eab-kid "YOUR_EAB_KID" --eab-hmac-key "YOUR_EAB_HMAC_KEY"

echo ">>>>>>> Ken: 3) Creating a certificate"

docker exec \
-e CF_Token="YOUR_CLOUDFLARE_TOKEN" \
-e CF_Email="YOUR_EMAIL" \
acme.sh \
--issue --dns dns_cf --dnssleep 60 -d "${DOMAIN}" -d "*.${DOMAIN}" --server zerossl

echo ">>>>>>> Ken: 4) Deploying the certificate"

# see cookie for your DSM from edge browser (if your DSM's IP is 192.168.1.100):
# edge://settings/cookies/detail?site=192.168.1.100

docker exec \
-e SYNO_Username="YOUR_DSM_USER_NAME" \
-e SYNO_Password="YOUR_DSM_PASSWORD" \
-e SYNO_Certificate="" \
-e SYNO_Scheme="https" \
-e SYNO_Port="5001" \
-e SYNO_DID='YOUR_DSM_DID_IF_YOU_USE_2FA' \
acme.sh \
--deploy --insecure -d "${DOMAIN}" -d "*.${DOMAIN}" \
--deploy-hook synology_dsm

echo ">>>>>>> Ken: Stopping acme.sh"

docker stop acme.sh

echo ">>>>>>> Ken: #### End of my SSL update script ####"
```

For information about **SYNO_DID** and deploying through **HTTPS**, check this:

https://github.com/acmesh-official/acme.sh/wiki/Synology-NAS-Guide#deploy-the-default-certificate

### Create a scheduled task

Open **Control Panel**, **Task Scheduler**, create a new **scheduled task**/**User-defined script**.

Name it as **acme**, run as **root**.

Schedule it to run **Repeat monthly**.

On **Task Settings**, enter this for the **User-defined script**:

```bash
bash /volume1/docker/acme/my_update_ssl.sh >>/volume1/docker/acme/log_acme/log.txt 2>&1
```

A note for the meaning:

it run the script, then redirect the output the the log.txt.

> That part is written to stderr, use 2> to redirect it. For example:
>
> foo > stdout.txt 2> stderr.txt
> or if you want in same file:
>
> foo > allout.txt 2>&1

> File descriptor 1 is the standard output (`stdout`).
> File descriptor 2 is the standard error (`stderr`).
>
> At first, `2>1` may look like a good way to redirect `stderr` to `stdout`. However, it will actually be interpreted as "redirect `stderr` to a file named `1`".
>
> `&` indicates that what follows and precedes is a *file descriptor*, and not a filename. Thus, we use `2>&1`. Consider `>&` to be a redirect merger operator.

https://stackoverflow.com/questions/818255/what-does-21-mean

https://stackoverflow.com/questions/6674327/redirect-all-output-to-file-in-bash
