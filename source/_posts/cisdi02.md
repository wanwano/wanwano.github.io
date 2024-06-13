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

![dahua show](/img/cisdi02/collabora.png)

# knowledge

- [Nginx](https://nginx.org/en/)



