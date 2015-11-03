# ProXPN OpenVPN Bash Client

[ProXPN](http://proxpn.com/) provides [OpenVPN](https://openvpn.net/) exit nodes all around the world and is a great service to use when connected to untrusted WiFi hotspots or a censored and monitored network. It's also very useful if you need to alter your geographic location to test web applications or services from different locales.

The problem with ProXPN, is that until recently they didn't provide or maintain a Linux client, so Linux users were left to sort out how to securely connect to ProXPN's OpenVPN servers themselves or stop using the service. This repo endeavored to fix that.

```
$ sudo proxpn

Welcome to the ProXPN OpenVPN Bash Client!

Which exit node would you like to use?
1) Chicago      5) Dallas   9) NYC        13) Miami
2) Sweden       6) BASIC    10) Stockholm 14) SanJose
3) Netherlands  7) London   11) Prague
4) Singapore    8) LA       12) Seattle

Select an exit node:9

Running:
/usr/bin/openvpn --config /etc/proxpn/proxpn.ovpn --remote se1.proxpn.com 443 --auth-user-pass /etc/proxpn/login.conf --auth-nocache

```

### Prerequisites

Because this repo is designed to enable secure and private communications, dependencies have been kept to an absolute minimum. All that is required to use this shell script is `openvpn` itself, and of course a ProXPN account. 

It's likely your system's package manager knows how to get and install `openvpn` already. [More details about installing `openvpn` can be found here](https://openvpn.net/index.php/open-source/documentation/howto.html#install).

**Debian/Ubuntu**

The official repositories have OpenVPN available so install it!

```
apt-get install openvpn
```

**RPM**

The official repositories have OpenVPN available so install it!

```
rpm -i openvpn
```

**Arch Linux**

The official repositories have OpenVPN available so [install it with pacman](https://wiki.archlinux.org/index.php/OpenVPN#Install_OpenVPN).

```
pacman -S openvpn
```

**OS X**

The simplest way to install OpenVPN in OS X is with [Homebrew](http://brew.sh/)

```
brew install openvpn
```

### Installing This Script

To use this script clone this repo or download the source from the releases page. The recommendation is to create a `proxpn` directory in `etc` and store the included `proxpn.ovpn` file there.
The main script should be given execute permissions `chmod +x proxpn` and copied to somewhere in your `$PATH`. The recommendation is to copy the script to `/usr/local/bin` or `/usr/bin`. This should allow you to run the command `proxpn` from any location.

This can now all be done using the provided installation script: `./install.sh`  

### Authentication Credentials

This script does not handle ProXPN authentication at all. When the user is prompted for their credentials that is coming entirely from `openvpn` itself. If you want to avoid entering your credentials at all you can create a file at `/etc/proxpn/login.conf` with your ProXPN username on one line and password on another. Again, this shell script does NOTHING with that file other than point OpenVPN to it. The specification for that file and the `--auth-user-pass` flag can be found in the [OpenVPN documentation](https://openvpn.net/index.php/open-source/documentation/manuals/65-openvpn-20x-manpage.html).

### Usage

This shell script will need to be run using `sudo` because it's establishing a VPN connection. You'll notice when installing ProXPN's client software on supported systems it will also request elevated access. While this isn't something to take lightly, the hope is this script is easy enough to digest that you feel confident and secure running it as `sudo`.  
If the script is run without `sudo` dry run mode is enabled, which does not attempt to establish a connection but prints an OpenVPN command to STDOUT that can be copied and used directly. Dry run mode can also be enabled by passing the flag `--dry-run`.

### How does this script work?

Bash is notorious for being difficult to read and understand. For this reason the code has been kept simple, brief, and is liberally commented. The following list describes, in plain English, what this script is doing.

  1. Check if openvpn is installed:
    - If it isn't, exit. 
    - If it is, store the path to the executable as the variable `OPENVPN`
  1. Setup constants which tell the program where to find the OpenVPN config file and where to look for an optional file containing the user's ProXPN credentials (by default it checks inside `/etc/proxpn`).
      - If no OpenVPN config file is found, exit. 
      - If no credentials file is found tell the user they're expected to login.
  1. Check if the command was run without `sudo` or if the flag `--dry-run` was passed. 
    - If it was, set the `dryRun` variable to `true`, causing the script to stop before attempting to open a connection to ProXPN and instead print an OpenVPN command to STDOUT.
  1. Setup a [bash `trap`](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_12_02.html) to catch if the user prematurely exits the script by pressing control-c.
    - If they do, make sure that `stty echo` has been called. This ensures that if user input was hidden (OpenVPN does this after prompting for a username and password) it doesn't continue to be hidden after the application is closed. This is a terminal specific issue.
  1. Setup an array containing all of the ProXPN exit node domain names as provided by ProXPN customer support
  1. Show the user all of the exit nodes available and ask them to choose one
    - If their response is valid, set the remote server 
    - If it isn't, exit.
  1. If in dry run mode, print the OpenVPN command to STDOUT and exit, otherwise attempt to establish a connection to ProXPN.
    - If the user hasn't provided a credentials file to `openvpn` and dry run mode was not enabled then they will be prompted for a password.
    - Terminate the VPN tunnel at anytime with control-c.

### Does this work for the free edition of ProXPN?

Yes, but non-paid ProXPN users will only be able to successfully connect to the rate limited `BASIC` exit node.
