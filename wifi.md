# Wireless network configuration


1. Check kernel messages for firmware being loaded

   ```
   # dmesg | grep firmware
     [   7.148259] 
     iwlwifi 0000:02:00.0: loaded firmware version 39.30.4.1 build 35138 op_mode iwldvm
   # dmesg | grep iwlwifi 
     [   12.342694] iwlwifi 0000:02:00.0: irq 44 for MSI/MSI-X
     [   12.353466] iwlwifi 0000:02:00.0: loaded firmware version 39.31.5.1 build 35138 op_mode iwldvm
     [   12.430317] iwlwifi 0000:02:00.0: CONFIG_IWLWIFI_DEBUG disabled
     ...
     [   12.430341] iwlwifi 0000:02:00.0: Detected Intel(R) Corporation WiFi Link 5100 AGN, REV=0x6B
   ```
   
2. Find the name of wireless interface and bring up before you can use iw
   ```
   # iw dev                 # also ip link
   # ip link set wlp3s0 up
   # ip link show wlp3s0    # to verify
   ```

3. Manual wireless management

   |                              |                     |
   | ---------------------------- |:-------------------:|
   | interface activation         | ip                  |
   | wireless connection manager  | iw + wpa_supplicant |
   | IP assigning                 | ip (or dhcpcd)      |
   ```
   # pacman -S iw wpa_supplicant
   ```

4. Access point discovery
   ```
   # iw dev wlp3s0 scan | less    # Important points to check: 
                                  # SSID, Security, Group,  Pairwise and Authentication
   ```

5. Make a configuration file for wpa_supplicant 

   **CUSTOMIZE to your convenience**  
   more samples: /etc/wpa_supplicant/*.conf
   ```
   # cat <<EOF > /etc/wpa_supplicant/JAZZTEL
   network={
       ssid="JAZZTEL"
       psk="mi%contraseña"
       proto=RSN
       key_mgmt=WPA-PSK
       pairwise=CCMP
       group=CCMP
   }
   EOF
   ```

6. Connect with wpa_supplicant
   ```
   # wpa_supplicant -B -i wlp3s0 -c /etc/wpa_supplicant/JAZZTEL.conf -D nl80211,wext
   ```

7. Check link status when connected to an AP
   ```
   # iw dev wlp3s0 link
   ```

8. Getting IP address for DHCP
   ```
   # dhcpcd wlp3s0
   # ping -c 3 www.google.com
   ```

9. Wpa_supplicant provides systemd services to start at boot
  ```
  # cp /etc/wpa_supplicant/JAZZTEL.conf /etc/wpa_supplicant/wpa_supplicant-wlp3s0.conf
  # systemctl enable wpa_supplicant@wlp3s0  # Accepts the interface name as an argument 
                                            # and starts the wpa_supplicant daemon for this interface.
                                            # It reads the configuration file in /etc/wpa_supplicant/wpa_supplicant-interface.conf.
  # systemctl enable dhcpcd@wlp3s0          # Enable a service to obtain an ip address for the particular interface
  ```




# Resources
[1] https://wiki.archlinux.org/index.php/Wireless_network_configuration  
[2] https://wiki.archlinux.org/index.php/WPA_supplicant  
