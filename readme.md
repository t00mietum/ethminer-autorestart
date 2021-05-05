# Readme<!-- omit in toc -->

## Table of contents<!-- omit in toc -->
- [Purpose](#purpose)
- [Contents](#contents)
- [Get, install, and configure](#get-install-and-configure)
	- [How your final folder structure should look](#how-your-final-folder-structure-should-look)
	- [Configure Cron](#configure-cron)
	- [First run](#first-run)
- [Warning](#warning)
	- [Physical damage](#physical-damage)
	- [These scripts run with root-level permissions](#these-scripts-run-with-root-level-permissions)
- [License](#license)
- [Donations](#donations)

## Purpose

This is an boot-time startul, automatic restart, and logging wrapper for the popular linux (and Windows) Ethereum GPU miner, [Ethminer](https://github.com/ethereum-mining/ethminer).

It's a small collection of simple bash scripts. It works whether you have dedicated miners, or a desktop/part-time miner.

## Contents

| Scriptname       | Description |
| :---             | :---        |
| `et`             | Run by `et-start` and hourly system `cron` - *not by user* (but you should inspect and make sure you understand *exactly* what it's doing). The execute bit is managed by `et-start` and `et-stop`. **You will update this script with your public Ethereum key, your rig ID, GPU type, and your mining pool address**. Otherwise, don't run directly.
| `et-interactive` | This is more like a typical user executable we're all familiar with: You run it. You see the output sroll by. You hit CTRL+Break, and it stops. The big difference is: it's actually running as a system process in the background, and gets restarted every hour. Even if your entire user session crashes, it keeps going. Only by pressing CTRL+Break in the system `screen` session named 'eth', will it stop, and disable hourly restarts.
| `et-start`       | Can be run manually by user, and optionally by boot-time system `cron` for servers or dedicated miners.
| `et-stop`        | Can be run manually by user to stop mining and disable hourly restarts, and optionally by boot-time system `cron` for desktop PCs when you don't want mining to start automatically.

## Get, install, and configure

```
cd $(mktemp -d)
git clone https://github.com/t00mietum/ethminer-autorestart
sudo mkdir -p /usr/local/bin/et
sudo mv ethminer-autorestart/* /usr/local/bin/et/
sudo chown -R root:root /usr/local/bin/et
sudo chmod 770 /usr/local/bin/et/*
sudo nano /usr/local/bin/et/et   ## Update with your eth public key, GPU type, rig id, pool address
```

Next, copy your ethminer `bin` directory, under the `et` directory. You'll neet sudo or root permissions to do so.

### How your final folder structure should look

```
/usr/local/bin/
	et/
		et
		et-interactive
		et-start
		et-stop
		readme.md
		bin/
			ethminer
			kernels/
				...
```

### Configure Cron

This is the heart of it. If mining was running (whether started by the user or the system), it will be restarted every hour.

If the user had stopped it, or it had never been started after boot, then it won't be automatically restarted.

Summertime heat-related schedules can also be configured.

Get started in a terminal prompt by typing:

`sudo crontab -e`

Then copy and paste the following:

```
## Enable one of (depending on your preference):
#@reboot                          /usr/local/bin/et/et-start   ## Start mining on powerup or reboot, e.g. for headless servers or dedicated miners
 @reboot                          /usr/local/bin/et/et-stop    ## Do not start mining on powerup or reboot, e.g. for desktops that people use

## Restart et every hour (but may not actually do anything every hour)
 0    *   *   *         *         /usr/local/bin/et/et         ## Will restart miner every hour if execute bit is enabled by et-start (either by @reboot and/or any time by user).

## Optionally configure for hot days (uncomment and edit as appropriate)
#0   12   *   JUN-SEP   *         /usr/local/bin/et/et-stop    ## Stop  mining at 12:00 in summer heat
#0   22   *   JUN-SEP   *         /usr/local/bin/et/et-start   ## Start mining at 22:00 in summer heat
#0   14   *   MAY,OCT   *         /usr/local/bin/et/et-stop    ## Stop  mining at 14:OO in May and October (less severe)
#0   20   *   MAY,OCT   *         /usr/local/bin/et/et-start   ## Start mining at 20:OO in May and October (less severe)
```

Save and exit.

### First run

From a terminal, execute `/usr/local/bin/et/et-interactive`. This will create the following symlinks that are in your path, so that you can execute them by name only (this is the only time your system is modified without your direct intervention):

* `/usr/local/bin/et-interactive`
* `/usr/local/bin/et-start`
* `/usr/local/bin/et-stop`

## Warning

### Physical damage

Mining involves a real risk of physical hardware damage or even fire, especially when forcing restarts after heat-related crashes. Please seriously investigate why restarts are needed. If crashes due to heat are the culprit, solve that problem before resorting to a band-aid like this.

And not to be the fire nanny, but please give consideration to running your mining rig(s) with working smoke alarms and automatic fire suppression systems nearby.

### These scripts run with root-level permissions

It should go without warning that you should never expose your system to scripts that run with administrator rights - *especially ones with access to your crypto key* (albeit public), without very careful inspection.

Make sure you know what every line of script is doing.

As for the question of "why" root-level access? Well it's the whole point of all - any - of this: so that cron (or init.d or SystemD or something) can restart mining periodically, and at system startup, indepentent of user login.

That said, if you don't understand Bash script, just try reading it line by line. If there's something you don't understand and can't easily figure out, you are strongly advised to not YOLO it and try running it anyway!

## License

Copyright (c) 2021 t00mietum.

License GPLv3+: GNU GPL version 3 or later, full text at:
    https://www.gnu.org/licenses/gpl-3.0.en.html

There is no warranty, to the extent permitted by law.

## Donations

If this improves your life, donations are welcome at:

* XMR: `44LiTd9eJZbBULNvzLsVUxYaJjcT7dxUPfBCwk8qbY1DN3nnsYBYomqWkhoe1t9RuAfGCJauZS2xw85PZPtPR5MR791cdqx`
* ETH: `0x01226C328A1d8085156b624D1DA78dC8ff954e8C`
* BTC: `bc1q2urqexl49w4t9e7xd9fwg7jpnkdunhle3chec4`
