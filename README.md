<p align="center"><img width="auto" height="90" src="https://github.com/oddlama/minecraft-server/blob/pages/docs/logo.png"></p>

<div align="center">

[![Landing Page](https://img.shields.io/badge/Landing_Page-informational.svg)](https://oddlama.github.io/minecraft-server)
[![MIT License](https://img.shields.io/badge/license-MIT-informational.svg)](./LICENSE)

</div>

# Minecraft Server Installer

This is a simple but fully-featured minecraft server installer for linux.
It should serve as an example of how to properly(TM) deploy a personal minecraft server with the following features:

- 🚀 Server starts automatically when players connect and shuts down when idle
- ⏱️ Utilizes [PaperMC](https://papermc.io) and [Aikar's JVM flags](https://aikar.co/mcflags.html) for maximum performance
- 🔒 Sandboxed execution with systemd, no docker
- 💾 Creates proper incremental world backups after each server stop
- 🖥️ Background console access via tmux (also remotely via ssh)
- 🔢 Account multiplexing allows a single account to have two or more player characters
- 🗺️ Awesome 3D online map using [BlueMap](https://bluemap.bluecolored.de/)
- 🔋 Single-command scripts to update server and plugins
- 🐙 Ready for configuration file tracking via git
- <img width="auto" height="20px" src="https://github.com/oddlama/vane/blob/main/docs/vane.png"> Includes vanilla enhancements by [vane](https://github.com/oddlama/vane)

## 🛠 Installation

To begin the automatic installation, simply run the provided bootstrap script.
Afterwards you can continue [configuring your server](#Server-Configuration),
or jump straight to the [Usage](#Usage) section if you are happy with the defaults.

```bash
curl -sL https://raw.githubusercontent.com/Krak3N22/minecraft-server/refs/heads/main/installer/bootstrap | sudo bash
# Connect to the console (Press Ctrl+b then d to detach again)
sudo minecraft-attach server
# Don't forget to foward or expose TCP ports 25565 (server), 25566 (multiplexer 1)
# and 8100 (online map). The map will be available under http://<your-ip>:8100
```

You may want to [review](https://github.com/Krak3N22/minecraft-server/blob/main/installer/bootstrap) the script before executing it.
In summary, the script will perform the following steps:

- Check whether all required tools are installed
- Create a new `minecraft` user for the server (if not existing)
- Update the server jars and download all plugins
- Install, enable and start the systemd services

## ⚙️ Server configuration

At this point, your proxy server is already running and the actual
server will be started once you connect to it. Now is the time to
review or change the server configuration. The main directory of your server
is `/minecraft/paper-minecraft-vane`. All files in that directory must be accessible to the `minecraft` user,
so before changing anything it is a good idea to open a terminal as the
minecraft user by executing `sudo runuser -u minecraft bash`.

The following sections are dedicated to things that you might want to configure now.
All other settings that were already changed by this script compared to the minecraft default
are listed in [Default Settings](#Default-Settings).

When you are happy with your configuration, continue to the [Usage](#Usage) section.

### 🌱 Seed

Before your server is started for the first time, you can can specify a seed via `level-seed=`
in `server.properties`. To find a good seed before generating your world, have a look
at [Cubiomes Viewer](https://github.com/Cubitect/cubiomes-viewer).

### 📜 Whitelist / Graylist

By default there is no protection enabled, everyone can join and play.
You can enable a classic whitelist with `whitelist on`.

Another option that vane provides is a feature similar to a graylist.
This allows anyone to connect to your server but in a *no touch, only look!* kind of way.
To modify anything about the world, a player must either be opped,
or be assigned to the user group with `perm add player_name user`, or any higher group.
To enable the graylist, set the following option:

```yaml
# server/plugins/vane-admin/config.yml
world_protection:
  enabled: true
```

Additionally, if you assign a player to the verified group with `perm add player_name verified`,
they may vouch for other users by using `/vouch other_player`. This will lift the other user into the `users` group.
Useful to give your friends the permission to invite other people they know. It will
be stored who vouched for whom.

### 🗒️ Serverlist text & icon

The text and icon in your server list now controlled by the proxy instead of your `server.properties`.
Edit `proxy/plugins/vane-velocity/config.toml` to change the text to your liking. You can also
set different texts based on whether the server is currently started or not.

To set a server icon, simply drop a file name `server-icon.png` in your `server/` directory,
next to where the `paper.jar` is.

## 🚀 Usage

In the following you will learn how to use the features of this deploy
to access the console, update your server among other things.

### 🔑 Accessing the server/proxy console

Access to your server console is crucial. The services keep both the proxy and server
console in the background at all times, so you can access them from any
terminal on your server (also remotely via ssh!).

```bash
sudo minecraft-attach server # Open the server console
sudo minecraft-attach proxy  # Open the proxy console
```

Once you execute one of the commands above, you will be presented
with the respective console. If that command fails, make sure the
system services are running! Press <kbd>Ctrl</kbd> + <kbd>b</kbd> followed by <kbd>d</kbd>
to leave the console. This will put it in the background again.

### 🗺️ 3D Online map (BlueMap)

The awesome 3D online map comes fully preconfigured. All you need to
do is open `http://<your-server-address>:8100` in your favourite browser,
when your server is online. Replace your-server-address with the IP or domain name
you use to connect in minecraft.

If you have an external webserver, BlueMap can be configured to be always available.

### 🔢 Account multiplexing

A multiplexer is an additional port for your server. When someone connects
via this port, they will be logged into a secondary player character. This also
works while being logged in on the main server. Very useful for account sharing or
to hand out spectator accounts. Just add a new serverlist entry for the multiplexer
and enjoy having multiple accounts!

Two accounts is not enough? Adding additional multiplexers is simple:

1. Forward or expose a new port. (e.g. 25567)
2. Add a new config for the multiplexer in `proxy/plugins/vane-velocity/config.toml`:
   ```
   [auth_multiplex.2]
   port = 25567
   allowed_uuids = [] # anyone may use this multiplexer
   ```
3. (Repeat for each additional multiplexer you want to add)

To disable this feature altogether, just remove all `auth_multiplex.*` sections from the config.

### 🔄 Updating the server

To update the server jars and all plugins, we first stop all services,
run the updater and then start them again. To do this, execute the
following commands as root:

```bash
systemctl stop minecraft-proxy minecraft-server    # Stop services
cd /minecraft/paper-minecraft-vane/deploy                       # Change into deploy directory
./update.sh                                        # Run update script
systemctl start minecraft-proxy minecraft-server   # Start services again
```

### 🔄 Updating the deploy script

Generally you shouldn't need to update the deploy script.
The only time it might be necessary is when one the included plugins changes
its download location. You will notice when that happens, as `./update.sh` will fail.
Other updates to this repository will most likely be minor changes.

To update, execute the following commands as root:

```bash
systemctl stop minecraft-proxy minecraft-server    # Stop services
cd /minecraft/paper-minecraft-vane/deploy                       # Change into deploy directory
git pull                                           # Get updates from upstream
./contrib/install.sh                               # Re-install the service files
systemctl start minecraft-proxy minecraft-server   # Start services again
```

### 🔌 Installing and removing plugins

Plugins are installed and updated by the `update.sh` scripts.
To add a new plugin, find a download link that always points to the latest version
and add an entry at the end of the respective script, similar to those that are already present.

For example to add worldguard, you add the following at the end of `server/update.sh`:
```bash
download_file "https://dev.bukkit.org/projects/worldguard/files/latest" plugins/worldguard.jar
```

To remove plugins, simply delete the jar file and remove the corresponding line in the
script. To remove a vane module, remove it from the list in the for loop.

### 🔐 Changing permissions plugin

By default, this setup uses a very lightweight permission plugin called `vane-permissions`.
If you want to use a different permission plugin, be sure remove `vane-permissions` from the
plugins as shown above and follow [this guide](https://github.com/oddlama/vane/wiki/Installation-Guide#3-give-permissions-to-players)
in order not to break vane with your new plugin.

### 💾 Restoring backups

Your server will automatically create an incremental backup of all three worlds and the plugin folder everytime the server is stopped.
You can view all the backups that have been created until now by executing the following commands as root:

```bash
cd deploy/server
rdiff-backup -l backups/world
```

Now if anything happens on your server and you want to revert to an older version,
you can do so by simply executing the following commands as root:

```bash
cd deploy/server
rm -rf world  # First delete what you want to restore
rdiff-backup -r 1B backups/world world # Restore state from the last backup.
# Repeat analogously for any other folders that you want to restore:
# You can restore world, world_nether, world_the_end, and plugins
```

The `1B` just refers to the last backup, `2B` would be the second last.
You can also pick any of the times listed by `rdiff-backup -l` from before (like
`2022-06-22T20:21:42+02:00`) or pick a relative time like `2D` (2 days ago).

If you want to reduce the size of your backups, try `rdiff-backup --remove-older-than 6M` to delete
any backups older than six months. You can also do a lot more specific things with rdiff-backup.
Visit [their website](http://rdiff-backup.nongnu.org/examples.html) for more information.

### 💾 Changing or disabling backups

To create backups, the service calls the `server/backup.sh` file automatically each time the server stops.
Feel free to adjust this script to your liking. To completely disable backups, replace the script's content with:

```bash
#!/bin/bash
exit 0
```

### 🐙 Tracking configuration with git

This project include a utility script called `contrib/organize_configs.sh`. If you execute it,
it will sort the keys in all your configuration files alphabetically so they can be tracked by git properly.
This is necessary as the server will rewrite the configuration files each time the server is started,
causing the entries to shift around unpredictably.

The `.gitignore` files are already setup so you will not accidentally commit your whole world
or some cache files. Only configuration files are considered by default.

## 🔧 Default settings

This project comes with a reasonable default configuration for paper (main server)
and velocity (proxy server). This ensures that autostarting and account multiplexing
work out of the box. Some of these configs depend on your choices in the bootstrap script,
denoted below by the *(asks)* prefix. These are the configuration defaults that differ
by default from a freshly generated configuration:

#### Proxy settings

- Configure the proxy server (online mode, autostart, ...)
- Enable one account multiplexer (second player character for each account)

#### Spigot settings

- Remove unnecessary aliases from `commands.yml`
- Tell PaperMC that a proxy is used.
- Don't have PaperMC restart the server on crash. The system service takes care of that.
- Prevent annoying infinite sound broadcasts (dragon death, end portal, wither).
- Change the *moved-too-quickly* threshold to be less aggressive (ensures smoother elytra flight).
- Allow players to see entities up to 512 blocks away.
- Lower xp and item merge radius for a more vanilla experience while still reducing lag.

#### PaperMC settings

- (asks) Allow TNT duping and bedrock removal.
- (asks) Enable Anti-XRAY.
- (asks) Replenish loot chests after 1-2 realtime days.
- (asks) Disable hopper item move event.

#### Vanilla settings

- Set difficulty to HARD
- Increase slots to 20
- (asks) Increase view distance to 15 chunks
- Increase entity broadcast range (allow players to see entities far away)
- Disable spawn protection (use better setting from vane-admin if you want this)
- Set online mode to false (this is checked by the proxy)
- Listen on port 25501 so proxy can connect (**do not** forward this port!)

## ⭕ Uninstalling

If you want to uninstall this server, simply execute the following commands:

```bash
# Disable & stop services
sudo systemctl disable --now minecraft-{proxy,server}
# Remove service files and attach script
sudo rm /lib/systemd/system/minecraft-{proxy,server}.service /usr/bin/minecraft-attach
# Remove user
sudo userdel -r minecraft
# Delete files in /minecraft/paper-minecraft-vane/
sudo rm -rf /minecraft/paper-minecraft-vane/
```

## 🛠️ Useful tools

- [Cubiomes Viewer](https://github.com/Cubitect/cubiomes-viewer) - To find a good world seed
- [MCASelector](https://github.com/Querz/mcaselector) - To trim e.g. unpopulated chunks
