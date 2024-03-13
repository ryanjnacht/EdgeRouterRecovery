# Edge Router Recovery

## These are the distilled instructions for recovering a Ubiquiti Edge Router via console cable with a new USB stick using macOS

### Hardware Requirements
1. Serial console cable
2. Ethernet cable
3. Ethernet adapter for Mac
4. New USB stick (at least 1Gb)

### Preparing the hardware
1. Open the Edge Router and replace the USB stick with your new USB stick

### Setting up tftpd and httpd
On macOS, we will use the default tftpd. The configuration is located at `/System/Library/LaunchDaemons/tftp.plist`

1. create default tftpd directory if it doesn't exist: 
	* `mkdir /private/tftpboot`
2. set local eth0 to `192.168.1.20`
3. start tftpd: 
	* `sudo launchctl load -F /System/Library/LaunchDaemons/tftp.plist`
4. launch a basic httpd from your `~/Downloads` directory:
	* `python3 -m http.server 80`

### Download Images
1. download [emrk-0.9c.bin](emrk-0.9c.bin) and place it in the tftpd root directory `/private/tftpboot`
2. download the [latest firmware](https://ui.com/download/software/) for your Edge Router ([ER-e100.v2.0.9-hotfix.7.5622762.tar](ER-e100.v2.0.9-hotfix.7.5622762.tar)) and place it in `~/Downloads`


### Recovery process
1. unplug the power from the edge router
2. connect a serial cable between the edge router console port and your mac
3. connect an ethernet cable between the edge router eth0 and your mac
4. open tty (your serial cable device path may vary)
	* `screen /dev/tty.usbserial-A50285BI 115200`

5. plug in the edge router and hold down the space bar. wait for booting to stop at a prompt (Octeon ubnt_e100#)
6. configure for tftp boot: 
	
	* `set ipaddr 192.168.1.20`
	* `set netmask 255.255.255.0`
	* `set serverip 192.168.1.10`
	* `set bootfile emrk-0.9c.bin`
	* `tftpboot`
	
7. boot into emrk
	* `bootoctlinux $loadaddr`

8. follow the EMRK prompts: 
> Enter 'Yes' to proceed, 'No' to reboot
> yes or no: **yes**
> 
> Do you want to configure network via DHCP?
> yes or no: **no**
> 
> Do you want to configure network statically?
> yes or no: **yes**
> Enter IPv4 address in CIDR format (e.g. 192.0.2.20/24): **192.168.1.20/24**
> Enter IPv4 gateway address: **192.168.1.10**
> Enter DNS server address: **192.168.1.10**

9. initiate emrk-reinstallation
	*  `emrk-reinstall`
	
	> **yes**
	> 
	> EdgeOS image url: **http://192.168.1.10/ER-e100.v2.0.9-hotfix.7.5622762.tar**
		
	* `reboot`
10. the device will reboot using the image from the usb disk. once it has fully booted, you can access the ui via `http://192.168.1.1` on eth0