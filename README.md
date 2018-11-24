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

# Testing WPA3 with Virtual Wi-Fi Interfaces

To **test WPA3 using virtual Wi-Fi interfaces**, you can execute the following commands. Note that
you will first have to disable your network manager (or disable Wi-Fi) so your operating system
will not interfere with our simulations.

	sudo modprobe mac80211_hwsim radios=3
	rfkill unblock wifi

	# Optionally kill other Wi-Fi clients the brute-for way:
	#sudo pkill wpa_supplicant

	# Open a new terminal, and in the directory hostapd execute:
	sudo ./hostapd hostapd_wpa3.conf -dd -K

	# Open another terminal, and in the directory wpa_supplicant execute:
	sudo ./wpa_supplicant -D nl80211 -i wlan1 -c supp_wpa3.conf -dd -K

To monitor traffic you can execute `sudo ifconfig hwsim0 up` and view traffic in wireshark.

It may be usefull to disable MAC address randomization when performing tests (or use `macchanger` to set a custom fixed MAC address).

