
⚠️ Prerequisites: 

SoftEther VPN Client (Linux; choose architecture accordingly)


https://www.softether-download.com/en.aspx?product=softether

Original author of the scripts is https://github.com/mfaizanse

----
Make sure that IP forwarding is enabled:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

If it returns `1`, we are good to go. If it is `0`, you need to edit `/etc/sysctl.conf` file. You can do this in `vi` or `gedit` or whatever you like:

```bash
#If you need to edit the sysctl.conf file
sudo vi /etc/sysctl.conf
```

Then uncomment the `net.ipv4.ip_forward=1` line, then save and close the file.

Next run the following command to apply the changes:

```bash
sudo sysctl -p
```

Now run the following command again, and check that `1` is returned (IP forwarding enabled).

```bash
cat /proc/sys/net/ipv4/ip_forward
```
---

Run the following command to find the local gateway of the system:

```bash
sudo netstat -rn
#the local gateway is the one where the Destination IP is 0.0.0.0
#it will probably be 192.168.0.1
```
---
Extract the SoftEther VPN Client `tar.gz` file into a location such as `~/softether`

```bash
mkdir ~/softether
tar -xf softether-vpnclient-blahblah.tar.gz -C ~/softether
```

Save the scripts you downloaded from github into `~/softether`

Open the `vpn_config` file, edit and save the following:

```
#where the extracted SoftEther VPN Client is
CLIENT_DIR=”/home/khadas/softether/vpnclient

#name for the virtual network interface
#after creating, it will be shown as vpn_nic1 if you run the ifconfig command
#you can set this as whatever you want, but need to remember it for future steps
NIC_NAME="nic1"

#name for the VPN. Do not confuse with username
ACCOUNT_NAME="yonseidronevpn"

#VPN Server Public IP Address
VPN_HOST_IPv4="165.132.XX.XX"

#The local gateway you found above
LOCAL_GATEWAY="192.168.0.1"
```

Open the `vpn-connect.sh` file, edit and save the following:

```
...
#Set IP routes for VPN
sudo ip route add $VPN_HOST_IPv4/24 via $LOCAL_GATEWAY
sudo ip route del default via $LOCAL_GATEWAY
sudo netstat -rn
...
#Refresh IP address info from VPN server
#sudo dhclient vpn_$NIC_NAME
```

We changed the subnet mask to /24 and disabled (commented) the `dhclient` command since we will be setting a static IP for this device.

📢 Ask the VPN server admin for your `IP Address` in line 37 of `vpn-connect.sh` file.

---
Check that the `.sh` files haven't been tampered with by a malicious third-party, then make them into executables with the `chmod` command:

```bash
chmod +x ./setup-client.sh
chmod +x ./remove-client.sh
chmod +x ./vpn-connect.sh
chmod +x ./vpn-disconnect.sh
```

Then run `setup-client.sh`:

```bash
#To run, run the following command
./setup-client.sh
```

Follow the steps shown on the terminal window.

For `Destination VPN Server Host Name and Port Number`, type the server IP address and port that the server is listening on:

```
165.132.XX.XX:AAAA
Where AAAA is the port number
```

~~For `Connecting User Name`, type `macbook` or whatever username you configured when you configured the VPN server.~~ 

📢 Ask the VPN server admin for your `username`


For `Used Virtual Network Adapter Name`, type `nic1` or whatever name you set above.

~~For `Password` type the password for the user you created (`passwd` in our example)~~


📢 Ask the VPN server admin for your `password`


For `Specify standard or radius`, type `standard`, as we used the standard authentication method.

---

When all steps are done, check that there were no errors and run the following command:

```bash
./vpn-connect.sh
```

To disconnect from the VPN, run the following command:

```bash
./vpn-disconnect.sh
```

To permanently sign out, first disconnect from the VPN and run the following command:

```bash
./remove-client.sh
```

---

For more fine tuned control, first set up the VPN client using the `[setup-client.sh](http://setup-client.sh)` script, then navigate to `~/softether/vpnclient`

Then start the vpn client by running the following command:

```bash
sudo ./vpnclient start
```

Then run the vpn configuration software:

```bash
./vpncmd
```

Type `2` to configure the VPN client, then type `HELP` to see available commands.
