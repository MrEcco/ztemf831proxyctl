# LTE-proxy on raspberry

Work with six ZTE MF831 modems

## Configure modem

This modem have very very very crazy thing: this have no local IP settings controls by default. For make it avaliable we need root access.

### Get root access

This modem have ADB shell, which can be enabled with browser link:

http://192.168.0.1/goform/goform_set_cmd_process?goformId=USB_MODE_SWITCH&usb_mode=6

In this link you see OK return status code.
After what ADB shell for modem is avaliable.

```bash
### Install ADB tools
apt-get install adb

### We need SSH things for it (see below)
ssh-keygen -b 2048 -f ztemf831
cat ztemf831     # private key
cat ztemf831.pub # public key

### Check devices
adb devices
## You see starting ADB-daemon, it PID and other daemon info
## and after you see one (or more) device

### Connect to root console
adb shell
## Very strange connection only for debug. Need create normal
## SSH access via priv/pub keys (password reset to unknown 
## after reboot)

### Change port for SSH (cannot disable iptables DROP rule)
vi /etc/rc5.d/S10dropbear
## For example, to 65222

### Copy priv and pub keys to root home dir
mkdir -p /home/root/.ssh
# public key
echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDDW<<<<CUTTED_STRING>>>>0oyaO5pfQb8N root@raspberrypi > /home/root/.ssh/authorized_keys
# private key
cat <<EOF > /home/root/.ssh/zte
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAw1owDVMDn7GR+VAifEO/pjEmmDycPOtEfmGMrSKvpfk4IcO2
............................ CUTTED ............................
DSG5WJGElQbqJDond306xdzi6yH1/P5eAX4BJ9llBUG593f0W8QapA==
-----END RSA PRIVATE KEY-----
EOF

### Reboot
reboot
```

After this, we have root access to modem via ssh (and if webface is broken - we need not to enable ADB via http endpoint)

```bash
ssh -i ztemf831 root@192.168.0.1
```

### Enable IP settings controls

Thanks for this link: https://blog.elevendroids.com/2014/06/changing-zte-mf823-4g-modem-ip-address/

```bash
### Connect to
ssh -i ztemf831 -p 65222 root@192.168.0.1

### After connect
vi /usr/zte_web/web/js/config/datacard/mf831/menu.js

```

Add to other 3-level routes

```json
{
  hash:'#router_setting',
  path:'adm/lan',
  level:'3',
  parent:'#device_setting',
  requireLogin:false,
  checkSIMStatus:false
}
```

```bash
### Reboot
reboot
```

After reboot local IP settings is avaliable
