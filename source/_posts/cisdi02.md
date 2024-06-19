---
title: cisdi02
date: 2024-06-13 16:40:04
tags: cisdi
---

# server
Mainly configure the network, the company with certified WiFi is really annoying, restart the server is very troublesome.

### Network configuration

There are two points that need to be addressed.

- the server can access the internet
- people in the group can easily connect to the server 

For the first point, I connected the server with a networked host box to the switch, configured **NAT** so that the box acted as a gateway, which was easy to implement.

```
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

I was going to build a **vpn** to implement the second point, but I ran into a lot of problems. I first used [WireGuard](https://www.wireguard.com/) to configure the box as a server and the server and PC as a client, but strangely the clients couldn't communicate with each other. So I deployed the [WireGuard](https://www.wireguard.com/) server to the cloud again, and this time it worked, but I ran into a more difficult problem. The WireGuard tunnel uses the UDP protocol for communication. However, the domestic carrier's QoS policy restricts the UDP rate limiting. As a result, the WireGuard tunnel cannot be used properly. Projects such as [Phantun](https://github.com/dndx/phantun) and [udp2raw](https://github.com/wangyu-/udp2raw) can be used to disguise UDP traffic, but with added complexity. Therefore, I considered using [OpenVPN](https://openvpn.net/), but currently encountered a problem that the client can connect to the vpn but cannot access the Internet normally after specifying the client ip. **#TODO**

# project

### Dahua
Connect with [Dahua ICC platform](https://open-icc.dahuatech.com/#/home) to achieve real-time preview and video playback functions. When the implementation can be a unified interface forward all requests can also be designed separately for each interface to do customized processing.

show:

![dahua show](/img/cisdi02/dahua.png)

### Collabora
Configuring [Collabora Online](https://www.collaboraoffice.com/) and implementing the [WOPI](https://learn.microsoft.com/en-us/microsoft-365/cloud-storage-partner-program/rest/) protocol enables online previews of various file types.

**WOPI REST API:**
```
    /**
     * wopi CheckFileInfo endpoint
     * <p>
     * Returns info about the file with the given document id.
     * The response has to be in JSON format and at a minimum it needs to include
     * the file name and the file size.
     * The CheckFileInfo wopi endpoint is triggered by a GET request at
     * https://HOSTNAME/wopi/files/<id>
     */
    @GetMapping("/wopi/files/{id}")
    public ResponseEntity checkFileInfo(@PathVariable String id, 
    @RequestParam("access_token") String token){}

    /**
     * wopi GetFile endpoint
     * <p>
     * Given a request access token and a document id, sends back the contents of the 
     * file.
     * The GetFile wopi endpoint is triggered by a request with a GET verb at
     * https://HOSTNAME/wopi/files/<id>/contents
     */
    @GetMapping("/wopi/files/{id}/contents")
    public ResponseEntity getFileContent(@PathVariable int id, 
    @RequestParam("access_token") String token){}

    /**
     * wopi PutFile endpoint
     * <p>
     * Given a request access token and a document id, replaces the files with the POST
     * request body.
     * The PutFile wopi endpoint is triggered by a request with a POST verb at
     * https://HOSTNAME/wopi/files/<id>/contents
     */
    @PostMapping("/wopi/files/{id}/contents")
    public ResponseEntity<Void> putFile(@PathVariable int id, 
    @RequestBody byte[] content){}
```

show:

![collabora show](/img/cisdi02/collabora.png)

### Collabora & proxy 

After the base demo, I needed to deploy the service to a multi-tier agent environment.

Firstly, collabora online is deployed in an Intranet server, I need to do [frp](https://github.com/fatedier/frp), and the public network frp server is configured as
```
frps.toml

bindAddr = "0.0.0.0"
bindPort = 7000
#kcpBindPort = 7000
quicBindPort = 7000

vhostHTTPPort = 8088
vhostHTTPSPort = 4433

transport.maxPoolCount = 2000
transport.tcpMux = true
transport.tcpMuxKeepaliveInterval = 60
transport.tcpKeepalive = 7200
transport.tls.force = false

webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin"
webServer.pprofEnable = false

log.to = "./frps.log"
log.level = "info"
log.maxDays = 3
log.disablePrintColor = false

auth.method = "token"
auth.token = "xxxxx"

allowPorts = [
  { start = 10001, end = 50000 }
]

maxPortsPerClient = 8
udpPacketSize = 1500
natholeAnalysisDataReserveHours = 168
```
And, the client likes:
```
frpc

serverAddr = "x.x.x.x"
serverPort = 7000
auth.method = "token"
auth.token = "xxxxx"

[[proxies]]
name = "web1_9e1645bc"
type = "http"
localIP = "x.x.x.x"
localPort = 9980
customDomains = ["x.x.x.x"]

[[proxies]]
name = "web2_9e1645bc"
type = "https"
localIP = "x.x.x.x"
localPort = 9980
customDomains = ["x.x.x4.x"]

[[proxies]]
name = "tcp1_9e1645bc"
type = "tcp"
localIP = "x.x.x.x"
localPort = 22
remotePort = 22222
```
Tip: Remember to close in time after use, it will be scanned by the company.

Next, we need a nameserver and an internal nginx server, each configured as follows:
```
internal nginx

collabora.conf

server {
    listen 80;
    charset UTF-8;
    client_max_body_size 75M;

    # static files
    location ^~ /browser {
      proxy_pass http://ip:port;
      #proxy_set_header Host $http_host;
    }
    
    # WOPI discovery URL
    location ^~ /hosting/discovery {
      proxy_pass http://ip:port;
      #proxy_set_header Host $http_host;
    }


    # Capabilities
    location ^~ /hosting/capabilities {
      proxy_pass http://ip:port;
      #proxy_set_header Host $http_host;
    }


    # main websocket
    location ~ ^/cool/(.*)/ws$ {
      proxy_pass http://ip:port;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "Upgrade";
      proxy_read_timeout 36000s;
      #proxy_set_header Host $http_host;
    }
    
    
    # download, presentation and image upload
    location ~ ^/(c|l)ool {
      proxy_pass http://ip:port;
      proxy_set_header Host $http_host; 
    }
    
    
    # Admin Console websocket
    location ^~ /cool/adminws {
      proxy_pass http://ip:port;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "Upgrade";
      proxy_read_timeout 36000s;
      #proxy_set_header Host $http_host;
    }
}
```

The nameserver listens for 443 requests and forwards the corresponding file storage service and the collabora service, which is similar to internal nginx.

For collabora office's configuration files, need to look at the following:
```
coolwsd.xml

Enter server_name in the conf file on the nameserver:

<server_name desc="External hostname:port of the server running coolwsd. If empty, it's derived from the request (please set it if this doesn't work). May be specified when behind a reverse-proxy or when the hostname is not reachable directly." type="string" default="">xxx.xxx</server_name>

Set to true:
<!-- SSL off-load can be done in a proxy, if so disable SSL, and enable termination below in production -->
<termination desc="Connection via proxy where coolwsd acts as working via https, but actually uses http." type="bool" default="true">true</termination>

Enter the address of the file storage service in the conf file of the nameserver:

<group>
    <host desc="Primary hostname to allow or deny." allow="true">https://xxx.xxx</host>
</group>
```

It's done.


# knowledge

- [Nginx](https://nginx.org/en/)



