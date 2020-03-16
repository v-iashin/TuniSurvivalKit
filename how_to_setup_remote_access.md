## Set up Remote Access

I assume that you have an Ubuntu 16.04 or 18.04 desktop (**host**) at your office and you would like to access it remotely from, let's say, your personal laptop (**client**). At this moment (13.03.2020), there are two ways to do it at Tampere University which we are aware of: 

1. Using `ssh-forward.cc.tut.fi` as a proxy server to connect to your host which is connected to `pit.cs.tut.fi`.
2. Connect both the host and client to VPN `staff-ras.vpn.tut.fi`. 

In both cases, the host machine will be reachable using a University-maintained laptop using University `TUNI-STAFF` WiFi or pre-installed VPN, for a self-maintained laptop you will need to do something extra.

## Connecting using a proxy server (`ssh-forward.cc.tut.fi`)

Pros:
- :+1: Only the `ssh` traffic is going through the University facilities unlike using a VPN.
- :+1: Stable. The IP of the host machine will not change after reconnection and the connection is lost rarely as it uses the wired connection.
- :+1: Fast. The machine is using a Gigabyte connection from the wall socket.

Cons:
- :hankey: You will need to contact [IT-Helpdesk](it-helpdesk@tuni.fi). This might take some time.
- :hankey: `ssh-forward.cc.tut.fi` doesn't support key-pair authentication. You can only login with passwords.
- :hankey: Different log-in procedure depending on the network you are using. One-step on `roam.fi/eduroam/TUNI-STAFF`, two-step on other networks.

### How to set up the host
1. Email [IT-Helpdesk](it-helpdesk@tuni.fi) to connect your office machine to `pit.cs.tut.fi` network. Specify the following things: a) the inventory number of the machine (on the sticker), b) MAC address of the socket in the machine you would like to use for the wired connection to the internet (you may have several Ethernet ports--you need only one), c) mention the Ethernet socket number from the wall that you will use. They will assign a fixed IP/FQDN and you will not need to type your credentials every 24 hours to have the internet connection.
2. At this point, you should have had received the response from [IT-Helpdesk](it-helpdesk@tuni.fi) and be able to connect to the internet using the socket you specified. If so, check your IP and type `host this-IP` to find out the FQDN. It should be something like `**********.pit.cs.tut.fi`.
3. Install `openssh-server` on your machine (host) (and on the client if you want). This will allow `ssh` connection to the machine. 
4. Next, make sure no WiFi connection connects automatically after the startup. Type `sudo nm-connection-editor` in terminal (or just go to `Edit connection` from the status menu on 16.04). Click on saved connections and go to `Preferences` (setting icon at the bottom) > `General` > uncheck the box.
5. Allow your `Wired connection` to automatically connect when available. `sudo nm-connection-editor`, go to `General` tab and make sure the box is checked.
6. Now you need to get access to `ssh-forward.cc.tut.fi`. Proceed to [tut.fi/omatunnus](https://www.tut.fi/omatunnus) (yes, `tut` not `tuni`) -> `Services` -> `System Access` (wait 10 sec) -> search for and select `ssh-forward.cc.tut.fi`. You will get the confirmation email shortly.
7. Initialize the two-step verification at `ssh-forward.cc.tut.fi`. For this, `ssh` with your TUNI credentials to `ssh-forward.cc.tut.fi` while being connected to one of the University networks (`roam.fi/eduroam/TUNI-STAFF` or a university VPN)--you cannot do it from any other network. Type `google-authenticator`. It will ask you several questions and show a QR code (resize your window to see it). Answer the questions as follows:
	- `Do you want authentication tokens...` -> y
	- `Do you want me to update your "/home/user/...` -> y
	- `Do you want to disallow multiple uses...` -> n
	- `By default, tokens are good for 30 seconds` -> n
	- `If the computer that you are logging into` -> y
8. Install `Google Authenticator` app for your smartphone. It is going to be used for two-step authentication when non-university network is used (connecting from home internet). You don't need an account to use this service if it will ask--just find a button to scan a QR code. Once it is done it will create an entry with a 6-digit passcode which changes every 30 secs.
9. To connect to your machine:
	- If you are on a University network (`roam.fi/eduroam/TUNI-STAFF` or VPN), `ssh` to `ssh-forward.cc.tut.fi`, type your TUNI credentials and `ssh` to your machine from that shell.
	- If you are using the non-University network, the procedure is the same but among TUNI credentials `ssh-forward.cc.tut.fi` will ask you for a `Verification Code`. For this, open the `Google Authenticator` app you install, and use the temporal code which is now shown there.

### Hint
You may find it useful to config the `ssh` connection (`~/.ssh/config`):
```
Host connection_name
  HostName ***********.pit.cs.tut.fi
  User user-for-***********.pit.cs.tut.fi
  ProxyCommand ssh your-tuni-username@ssh-forward.cc.tut.fi -W %h:%p
```
After doing this, you will be able to do `ssh connection_name` to `ssh` directly to `*********.pit.cs.tut.fi`. It is also useful if you are using `VSCode` or any other text editor which has a remote development option.

## Connecting using a proxy server (`staff-ras.vpn.tut.fi`)

### Pros:
- :+1: Quick solution. All you need is your TUNI credentials.

### Cons:
- :hankey: Your host machine should have a WiFi antenna.
- :hankey: Does not work on macOS. PPTP is not supported on macOS anymore.
- :hankey: Unstable. Sometimes WiFi or VPN connection on the host fails. So, the IP you want to `ssh` to will change and you will need to find out this IP.
- :hankey: Slow. For instance, `roam.fi/eduroam` without the VPN is \~20MB/s, it is going to be \~5MB/s with the VPN.
- :hankey:/:+1: Traffic goes through University. Remember, your personal laptop (client) is also connected to the VPN.
- :hankey: Credentials can be seen for `sudo`ers at the machine (host). Mind this if the machine is used by multiple users.

### How to set up the host:

1. Install `openssh-server` on your machine (host) (and on the client if you want). This will allow `ssh` connection to the machine. On Ubuntu 16.04 you may need to install things for PPTP: `sudo apt-get install network-manager-pptp pptp-linux`.
2. If it isn't already, connect to any available WiFi. If you use `roam.fi/eduroam` you will need to select `No CA certificate is required` and type your TUNI credentials as well as to `Store the password for all users` (in the password field) in `WiFi Security` tab in the WiFi connection settings. Also, make sure to allow this WiFi connection to connect automatically (in `General`).
3. Add new PPTP VPN connection (`sudo nm-connection-editor` -- there or if you are on 16.04 go to `Edit connection` from the status menu). Use `staff-ras.vpn.tut.fi` as a gateway and your TUNI credentials, select `Store the password for all users`. Click `Advanced`. There uncheck everything except for `MSCHAPv2` in authentication methods. Check `Use Point-to-Point encryption (MPPE)`, `Allow BSD data compression`, `Allow Deflate data compression`, `Use TCP header compression` other boxes left unchecked.
4. Next, prohibit `Wired connections` from connecting automatically after the startup. Type `sudo nm-connection-editor` in terminal (or just go to `Edit connection` from the status menu on 16.04). Click on wired connections and go to `Preferences` (setting icon at the bottom) > `General` > uncheck the box. Repeat for another connection if you have more than two Ethernet ports;
5. Allow your WiFi connection to automatically connect to VPN. `sudo nm-connection-editor`, go to `General` tab and check the box and select the VPN you setup before.
6. If you did everything correctly you should be able to connect to WiFi and VPN, your IP and your host will have IP `130.230.89.*`.
7. On your personal laptop:
	- If you have a University-maintained laptop, connect to TUNI (TUT) VPN and that is all;
	- If you have a self-maintained computer, repeat the same steps as on the host machine;
8. When both your office machine (host) and personal laptop (client) are connected to the VPN. You may `ssh` your machine.

### What to do if the machine got disconnected?
Well, this is not a stable solution. When WiFi or VPN connection is lost, IP will be different after reconnection. If you are lucky enough it will at least assign the new IP to your machine (host). In such cases, you may find it using `nmap -sn 130.230.89.0/24` which will scan full subnetwork and find all available IPs there. Try to `ssh` to each of them with your credentials until you find yours. If the new IP wasn't assigned, i.e. it couldn't reconnect to WiFi/VPN, you need to go physically there and reconnect it.

Some useful commands
- `nmcli connnection show` (will show the status of the networks you have);
- `nmcli connnection up id wifi-or-vpn-name` (will connect the machine to `wifi-or-vpn-name` network, `down` will disconnect)




