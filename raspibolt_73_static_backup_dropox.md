---
layout: default
title: Static Channel Backups
parent: Bonus Section
nav_order: 45
has_toc: false
---
## Bonus guide: Automatic backup of static.channel to your Dropbox
*Difficulty: easy*

Since lnd V0.6, Static Channel Backups (SCB) is supported. In a nutshell, every time a channel changes, lnd writes a new copy of the channels.backup file. For more details, see v0.6-beta Release Notes
This guide explains one way to automatically upload the channels.backup file on changes, to your Dropbox
The following scripts were created by [Vindard](https://github.com/vindard)

#### *Risk : Minimal* 
The channels.backup file is encrypted so that it is safe to transmit over the Internet and to store on (e.g.) a cloud server.

#### *Requirements: lnd version higher than 0.6*



### Setup Dropbox API Key
    
In your web browser, do the following:

1. Go to https://www.dropbox.com/developers/apps/create and sign in

1. Choose **Scoped access**

    ![Dropbox 1](https://github.com/d-hoffi/RaspiboltGerman/blob/main/images/dropbox1.png)

1. Choose **App Folder**

    ![Dropbox 2](https://github.com/d-hoffi/RaspiboltGerman/blob/main/images/dropbox2.png)

1. Name your app and click **Create App** to proceed

    ![Dropbox 3](https://github.com/d-hoffi/RaspiboltGerman/blob/main/images/dropbox3.png)

1. On the settings page for your new app, scroll down to **OAuth 2** and change the **Access token expiration** to `No expiration`

    ![Dropbox 4](https://github.com/d-hoffi/RaspiboltGerman/blob/main/images/dropbox4.png)

1. On the **Permissions* tab, check the boxes that is look the same like in the picture below. You can click ***Generate** in the **Settings** tab after this step. You will now see a string of letters and numbers appear. This is your **Api Token**. Copy this token and keep it safe for the next steps. This api token will be referenced as `<dropbox-api-token>` in the next step.

    ![Dropbox 5](https://github.com/d-hoffi/RaspiboltGerman/blob/main/images/dropbox5.png)


### Preparation on the RaspiBolt
As user "admin", download the script, make it executable and move to the global bin folder.

```bash
$ cd /home/admin/download/
$ wget https://gist.githubusercontent.com/vindard/e0cd3d41bb403a823f3b5002488e3f90/raw/4bcf3c0163f77443a6f7c00caae0750b1fa0d63d/lnd-channel-backup.sh

# check script & modify with your <dropbox-api-token> (third line)
$ sudo nano lnd-channel-backup.sh

$ sudo chmod +x lnd-channel-backup.sh
$ sudo mv lnd-channel-backup.sh /usr/local/bin
```

### Setup script as `systemd` service (start automatically on system startup)

`$ sudo nano /etc/systemd/system/backup-channels.service`

```ini
[Service]
ExecStart=/usr/local/bin/lnd-channel-backup.sh
Restart=always
RestartSec=1
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=backup-channels
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

Start

`$ sudo systemctl start backup-channels`

Monitor

`$ sudo journalctl -fu backup-channels`

Run at boot

`$ sudo systemctl enable backup-channels`

### Test 
You will now cause the channel.backup to change and see if the copy gets uploaded to your webserver.  

`$ sudo touch /home/bitcoin/.lnd/data/chain/bitcoin/mainnet/channel.backup`  

Logon to your dropbox and see if you have a new file.  

------

<< Back: [Bonus guides](raspibolt_60_bonus.md) 

