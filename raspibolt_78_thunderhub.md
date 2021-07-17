---
layout: default
title: Upgrade External Drive
parent: Bonus Section
nav_order: 140
has_toc: false
---
## Bonus Guide: Install Thunderhub

*Difficulty: medium*

This is a guide for the installation of [thunderhub](https://www.thunderhub.io/), manage your Lightning Node through a WebUI.

![](images/75_thunderhub.png)

---
## Preparations

### Check NodeJS Version

* Starting with user "admin", we switch to user "root" and check if [Node JS](https://nodejs.org) is installed already. 
  
  ```
  $ sudo su
  $ nodejs --version
  > v12.22.1
  $ exit
  ```
### Install NodeJS

* If that is not the case, we will install it. 
  We'll use version 12 which is the most recent stable one. Then, exit the "root" user session.

  ```
  $ curl -sL https://deb.nodesource.com/setup_12.x | bash -
  $ exit
  ```
* Install NodeJS using the apt package manager.

  ```
  $ sudo apt-get install nodejs
  ```

### Firewall 

* Configure firewall to allow incoming HTTP requests from your local network to the web server.

  ```
  $ sudo ufw allow from 192.168.0.0/16 to any port 3010 comment 'allow ThunderHub from local network'
  $ sudo ufw status
  ```

## ThunderHub

### Installation

We do not want to run the thunderhub code alongside `bitcoind` and `lnd` because of security reasons.
For that we will create a separate user and we will be running the code as the new user.
We are going to install thunderhub in the home directory since it doesn't take much space.

* Create a new user with  your `password[A]`. The new user needs read-only access to the `tls.cert` and our `admin.macaroon`, 
  so we add him to the "bitcoin" group. Open a new session.

  ```
  $ sudo adduser thunderhub
  $ sudo adduser thunderhub bitcoin
  $ sudo su - thunderhub
  ```

* Download the source code directly from GitHub and install all dependencies using NPM.

```
$ git clone https://github.com/apotdevin/thunderhub.git
$ cd thunderhub
$ npm install
$ npm run build
```

* Cancel the process with `Ctrl`-`C`.

### Configuration

* Edit the configuration file.

  ```
  $ nano /home/thunderhub/thunderhub/.env
  ```

* Uncomment the following lines:

  ```
  # -----------
  # Server Configs
  # -----------
  LOG_LEVEL='debug'

  # -----------
  # Interface Configs
  # -----------
  THEME='dark'

  # -----------
  # Account Configs
  # -----------
  ACCOUNT_CONFIG_PATH='/home/thunderhub/thubConfig.yaml'
  ```
* Edit your `thubConfig.yaml`. Change your `accountpassword`.

  ```
  cd
  nano thubConfig.yaml 
  ```
  ```
  masterPassword: 'PASSWORD' # Default password unless defined in account
  accounts:
    - name: 'RaspiBolt'
      serverUrl: '127.0.0.1:10009'
      macaroonPath: '/home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon'
      certificatePath: '/home/bitcoin/.lnd/tls.cert'
      password: 'accountpassword'
  ```

## First Start

Test starting thunderhub manually first to make sure it works.

* Let's do a first start to make sure it's running as expected.
  Make sure we are in the thunderhub directory and start the web server.

 ```
  $ cd ~/thunderhub
  $ npm run start -- -p 3010
  ```

* Now point your browser to `http://raspibolt.local:3010` (or whatever you chose as hostname) or the ip address (e.g. `http://192.168.0.20:3010`).
  You should see the home page of ThunderHub.

* If you see a lot of errors on the RaspiBolt command line, then you have to change file permissions maybe,  
  because thunderhub can't "access" the `.lnd` directory.

* Stop ThunderHub in the terminal with `Ctrl`-`C` and exit the "thunderhub" user session.

  ```
  $ exit
  ```

### Autostart on boot

Now we'll make sure ThunderHub starts as a service on the Raspberry Pi so it's always running.
In order to do that we create a systemd unit that starts the service on boot directly after LND.

* As user "admin", create the service file.

  ```
  $ sudo nano /etc/systemd/system/thunderhub.service
  ```

* Paste the following configuration. Save and exit.

  ```
  # RaspiBolt: systemd unit for Thunderhub
  # /etc/systemd/system/thunderhub.service

  [Unit]
  Description=Thunderhub
  Wants=lnd.service
  After=network.target lnd.service

  [Service]
  WorkingDirectory=/home/thunderhub/thunderhub
  ExecStart=/usr/bin/npm run start -- -p 3010
  User=thunderhub
  Restart=always
  TimeoutSec=120
  RestartSec=30
  StandardOutput=null
  StandardError=journal

  [Install]
  WantedBy=multi-user.target
  ```

* Enable the service, start it and check log logging output.

  ```
  $ sudo systemctl enable thunderhub.service
  $ sudo systemctl start thunderhub.service
  $ sudo journalctl -f -u thunderhub
  ```

* You can now access ThunderHub from within your local network by browsing to <http://raspibolt.local:3010> (or your equivalent ip address).

### Remote access over Tor (optional)

Do you want to access ThunderHub remotely?
You can easily do so by adding a Tor hidden service on the RaspiBolt and accessing ThunderHub with the Tor browser from any device.

* Add the following three lines in the section for "location-hidden services" in the `torrc` file.
  Save and exit.

 ```
  $ sudo nano /etc/tor/torrc
  ```

  ```
  HiddenServiceDir /var/lib/tor/thunderhub
  HiddenServiceVersion 3
  HiddenServicePort 80 127.0.0.1:3010
  ```

* Restart Tor and get your connection address.

  ```
  $ sudo systemctl restart tor
  $ sudo cat /var/lib/tor/thunderhub/hostname
  > abcdefg..............xyz.onion
  ```

* With the [Tor browser](https://www.torproject.org), you can access this onion address from any device.
  Please be aware that this access is not password protected and should not be shared widely.

**Congratulations!**
You now have the BTC RPC Explorer running to check the Bitcoin network information directly from your node.

## Upgrade

Updating to a [new release](https://github.com/apotdevin/thunderhub/releases) should be straight-forward.

* From user "admin", stop the service and open a "btcrpcexplorer" user session.

  ```
  $ sudo systemctl stop thunderhub
  $ sudo su - thunderhub
  ```

* Fetch the latest GitHub repository information and check out the new release:

  ```
  $ cd thunderhub
  $ git pull
  $ npm install
  $ npm run build
  $ exit
  ```

* Start the service again.

  ```
  $ sudo systemctl start thunderhub
  ```
---

Next: [Bonus guides >>](raspibolt_60_bonus.md)
