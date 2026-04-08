# Practice: Name resolution for containers

## Aardvark-DNS

Starting with Podman 4.0, DNS-based name resolution between containers is built in through **Aardvark-DNS**. Aardvark-DNS is an authoritative DNS server that runs automatically when you create a custom network. Containers on the same custom network can resolve each other by name — no additional plugins or manual setup required.

{% hint style="info" %}
DNS name resolution works for both rootful and rootless containers on custom networks (created with `podman network create`). It does **not** work on the default `podman` network.
{% endhint %}

1\. Create a new network using `podman network create`.

```
$ podman network create mynetwork 
mynetwork
```

2\. Verify that DNS is enabled on the new network

```
$ podman network inspect mynetwork | grep dns_enabled
          "dns_enabled": true,
```

3\. Create an nginx container on mynetwork

```
$ podman run -dt --name web --network mynetwork docker.io/library/nginx
3471dd9b4c4e681508d0c6de9fd2093eb12cf2affb259925c7da69ec21d3c554
```

4\. Try accessing nginx using the container name from another container on the same network

```
$ podman run -it --rm --network mynetwork docker.io/alpine sh

/ # wget -qO- http://web
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

5\. You can also verify DNS resolution works with nslookup

```
/ # nslookup web
Server:		10.89.0.1
Address:	10.89.0.1:53

Name:	web.dns.podman
Address: 10.89.0.2
```

The domain `dns.podman` is the default DNS zone used by Aardvark-DNS for container name resolution.

_**Congratulations! that concludes the exercise.**_
