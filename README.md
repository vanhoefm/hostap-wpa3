# Prerequisites

First install some dependencies:

	# Kali Linux and Ubuntu:
	sudo apt-get update
	sudo apt-get install libnl-3-dev libnl-genl-3-dev libnl-route-3-dev libssl-dev \
		libdbus-1-dev git pkg-config build-essential macchanger net-tools python3-venv \
		aircrack-ng rfkill


# Compiling the AP and Client

First compile hostapd:

	cd hostapd
	cp defconfig .config
	make -j 2
	cd ..

Now compile `wpa_supplicant`:

	cd wpa_supplicant
	cp defconfig .config
	make -j 2
	cd ..


# Configuring SAE-PK

First generate a private key:

	openssl ecparam -name prime256v1 -genkey -noout -out example_key.der -outform der

Now derive the password from it:

	cd hostapd
	make sae_pk_gen
	./sae_pk_gen example_key.der 3 SAEPK-Network

Example output:

	...
	sae_password=2udb-slxf-3ij2|pk=04e8aad54d1a121955e8703d1dfa115e:MHcCAQEEIKMP3SZEAlW9rSwTFsaR/sEyX963opsOo2QYe4G8Kcl+oAoGCCqGSM49AwEHoUQDQgAE4GuxyTkKNt0MEispu/XPxImInj+tl2ri/Jfu2mOQKb1TdNHSPs6UP+rxv5OWnezhOpjpD63Y+zjjz1yk7/iF7g==
	# Longer passwords can be used for improved security at the cost of usability:
	# 2udb-slxf-3ijn-y65k
	# 2udb-slxf-3ijn-y65x-vr2e
	# 2udb-slxf-3ijn-y65x-vr2i-6qob
	...

The example config files `hostapd_sae_pk.conf` and `supp_saepk.conf` can be used to create the SAE-PK network.


# Testing Using Virtual Interface

## 1.  Create Virtual Wi-Fi Interface

First create the virtual interfaces

	sudo modprobe mac80211_hwsim radios=3


## 2. Disable Wi-Fi in the Network Manager

Disable Wi-Fi in the network manager so your operating system won't interfere with our Wi-Fi tests.

Alternatively, you can blacklist the MAC address of your
Wi-Fi dongle so that Linux will automatically ignore the Wi-Fi dongle. This is done by adding
the [following lines](https://wiki.archlinux.org/index.php/NetworkManager#Ignore_specific_devices)
to the file `/etc/NetworkManager/NetworkManager.conf`:

	[keyfile]
	unmanaged-devices=mac:02:00:00:00:00:00

Replace `02:00:00:00:00:00` with the MAC addess of your (virtual) Wi-Fi dongle and then reboot Kali.


## 3. Start the client and AP

	# Manually enable Wi-Fi for our client and AP
	sudo rfkill unblock wifi

	# Optionally kill other Wi-Fi clients the brute-force way:
	#sudo pkill wpa_supplicant

	# Open a new terminal, and in the directory hostapd execute:
	sudo ./hostapd hostapd_wpa3.conf -dd -K

	# Open another terminal, and in the directory wpa_supplicant execute:
	sudo ./wpa_supplicant -D nl80211 -i wlan1 -c supp_wpa3.conf -dd -K

To monitor traffic you can execute `sudo ifconfig hwsim0 up` and view traffic in wireshark.

It may be usefull to disable MAC address randomization when performing tests (or use `macchanger` to set a custom fixed MAC address).

