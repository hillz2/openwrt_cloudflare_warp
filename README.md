
# How to use Cloudflare WARP on OpenWrt to bypass DPI (Deep Packet Inspection)

This tutorial was created mainly for Indonesian users, the government blocks some websites with DPI so simply changing the DNS doesn't work anymore. This is what I have:

> Router: GL.iNet 6416\
Firmware: OpenWrt 18.06.2\
Modem: Huawei E3372 HiLink ( With IP: 192.168.8.1)\
ISP: Tri Indonesia\
PC: Manjaro Linux (This doesn't really matter what you have)
> 
1. On your PC, download the appropriate wgcf binary release from Github  [https://github.com/ViRb3/wgcf](https://github.com/ViRb3/wgcf)  if you are using Linux the linux-amd64 binary is your best bet. Make sure to replace binary-release with the actual file name of the downloaded file
2.  Make the binary executable with: chmod +x binary-release
3.  Run ./binary-release register
4.  Accept terms and conditions
5.  Now run ./binary-release generate
6. You'll get **wgcf-profile.conf** file, which you'll need to setup wireguard on  your OpenWrt router. The file should look like this:

> [Interface]\
PrivateKey = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\
Address = 100.16.0.2/32\
Address = fddd:5ca1:ab1e:8daf:209d:9414:d1e0:5d2c/128\
DNS = 1.1.1.1\
MTU = 1280\
[Peer]\
PublicKey = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\
AllowedIPs = 0.0.0.0/0\
AllowedIPs = ::/0\
Endpoint = engage.cloudflareclient.com:2408
>
7. Now on your OpenWrt router do: **opkg update && opkg install wireguard wireguard-tools luci-proto-wireguard**
8. Edit your /etc/config/network and append the following lines, make sure to match the *private_keys* etc with the **wgcf-profile.conf** file that you have:

config interface 'Cloudflare'\
&nbsp;&nbsp;&nbsp;&nbsp;option proto 'wireguard'\
&nbsp;&nbsp;&nbsp;&nbsp;option private_key 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'\
&nbsp;&nbsp;&nbsp;&nbsp;list addresses '100.16.0.2/32'\
&nbsp;&nbsp;&nbsp;&nbsp;list addresses 'fddd:5ca1:ab1e:8129:b248:d4f:3f37:7fbe/128'\
&nbsp;&nbsp;&nbsp;&nbsp;option mtu '1280'\
&nbsp;&nbsp;&nbsp;&nbsp;option dns '1.1.1.1'\
\
config wireguard_Cloudflare\
&nbsp;&nbsp;&nbsp;&nbsp;option public_key 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'\
&nbsp;&nbsp;&nbsp;&nbsp;option endpoint_host 'engage.cloudflareclient.com'\
&nbsp;&nbsp;&nbsp;&nbsp;option endpoint_port '2408'\
&nbsp;&nbsp;&nbsp;&nbsp;list allowed_ips '0.0.0.0/0'\
&nbsp;&nbsp;&nbsp;&nbsp;list allowed_ips '::/0'\
&nbsp;&nbsp;&nbsp;&nbsp;option route_allowed_ips '1'\
\
config route 'route_wireguard'\
&nbsp;&nbsp;&nbsp;&nbsp;option interface 'Cloudflare'\
&nbsp;&nbsp;&nbsp;&nbsp;option target '0.0.0.0/0'\
&nbsp;&nbsp;&nbsp;&nbsp;option gateway '192.168.8.1' # This is the HiLink IP on my modem\
&nbsp;&nbsp;&nbsp;&nbsp;option metric '1024'\
\
config route 'route_bimatri' # This configuration is optional\
&nbsp;&nbsp;&nbsp;&nbsp;option interface 'HiLink' # Match this with the name of your hilink interface, mine is 'HiLink'\
&nbsp;&nbsp;&nbsp;&nbsp;option target '103.10.66.0/24' # This is the IP of bima.tri.co.id\
&nbsp;&nbsp;&nbsp;&nbsp;option option netmask '255.255.255.0'\
&nbsp;&nbsp;&nbsp;&nbsp;option gateway '192.168.8.1' # This is the HiLink IP on my modem\
&nbsp;&nbsp;&nbsp;&nbsp;option metric '1024'

9. Now do **/etc/init.d/network restart** 
10. Login to Luci WebUI. Go to Network > Interfaces and connect your Cloudflare Interface, if you're connected successfully, your Cloudflare interface should look like this:

![enter image description here](https://i.ibb.co/C685QtH/2022-04-10-192925-919x143-scrot.png) 

Your routing table should look like this:

![enter image description here](https://i.ibb.co/ysXtCtg/2022-04-13-144539-590x140-scrot.png) 

Now you should be able to access blocked websites like reddit.


References: \
https://www.reddit.com/r/openwrt/comments/kgk5r1/comment/ggfqvhe/?utm_source=share&utm_medium=web2x&context=3 \
https://openwrt.org/docs/guide-user/network/routing/routes_configuration
