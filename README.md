# BBB Persian

A short guide for how to setup a BigBlueButton server (without Greenlight) and make it suitable for Persian language

## Installation

1. Get an Ubuntu 20.04 Server.

2. Add Proxy to apt:

Add the following line to `/etc/apt/apt.conf.d/proxy.conf`:

```
Acquire::http::Proxy::download.docker.com "http://fodev.org:8118/";
Acquire::http::Proxy::repo.mongodb.org "http://fodev.org:8118/";
Acquire::http::Proxy::ubuntu.bigbluebutton.org "http://fodev.org:8118/";
```

3. Install Docker:

```shell
sudo apt update
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL -x "http://fodev.org:8118" https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. Add proxy to Docker:

```shell
sudo mkdir -p /etc/systemd/system/docker.service.d
```

Add the following lines to `/etc/systemd/system/docker.service.d/http-proxy.conf`:

```
[Service]
Environment="HTTPS_PROXY=http://fodev.org:8118"
```

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

5. Install BigBlueButton:

```shell
wget https://raw.githubusercontent.com/bigbluebutton/bbb-install/v2.7.x-release/bbb-install.sh

sudo bash bbb-install.sh -s -- -v focal-270 -s YOUR_SERVER_DOMAIN -e YOUR_EMAIL_ADDRESS -w
```

6. (Optional) Set secret:

```shell
sudo bbb-conf --setsecret YOUR_SECRET
```

7. Check the status of BieBlueButton:

```shell
sudo bbb-conf --status
```

## Changing the default locale to Persian

1. In the file `/usr/share/meteor/bundle/programs/server/assets/app/config/settings.yml`, change the value of `overrideLocale` to `fa`.

2. Restart BBB using `sudo bbb-conf --restart`.

## Removing welcome message footer text

1. In the file `/usr/share/bbb-web/WEB-INF/classes/bigbluebutton.properties`, set `defaultWelcomeMessageFooter` to `<div></div>`.

2. Restart BBB using `sudo bbb-conf --restart`.

## Removing the default presentation

1, Add the following line to `/etc/bigbluebutton/bbb-web.properties`:

`beans.presentationService.defaultUploadedPresentation=0`

2. Restart BBB using `sudo bbb-conf --restart`.

## Changing the font

1. Upload font files and styles to `/var/www/bigbluebutton-default/assets/images` such that the `/var/www/bigbluebutton-default/assets/images/css/style.css` should exist.

2. In the file `/usr/share/meteor/bundle/programs/server/assets/app/config/settings.yml`, change the value of `customStyleUrl` to `https://yourdomain.com/images/css/style.css`.

3. Restart BBB using `sudo bbb-conf --restart`.

## Enabling video recordings

1. In the file `/usr/local/bigbluebutton/core/scripts/bigbluebutton.yml`, change the following part:

```yml
steps:
  archive: "sanity"
  sanity: "captions"
  captions: "process:presentation"
  "process:presentation": "publish:presentation"
```

to this:

```yml
steps:
  archive: "sanity"
  sanity: "captions"
  captions:
    - "process:presentation"
    - "process:video"
  "process:presentation": "publish:presentation"
  "process:video": "publish:video"
```

2. Restart BBB using `sudo bbb-conf --restart`.

## Keep learning dashboard data permanently

1, Add the following line to `/etc/bigbluebutton/bbb-web.properties`:

`learningDashboardCleanupDelayInMinutes=0`

2. Restart BBB using `sudo bbb-conf --restart`.
