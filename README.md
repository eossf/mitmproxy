# Mitmproxy 
This is the fork of official mitmproxy github page.

Mitmproxy is a forward and reverse proxy, launched by default on port 8080.
(Ubuntu version)

## Setup the forward proxy

### 1 Install and use as transparent LOCAL (i.e. forward proxy)

````bash
# installation 
wget https://snapshots.mitmproxy.org/7.0.3/mitmproxy-7.0.3-linux.tar.gz
tar txvf mitmproxy-7.0.3-linux.tar.gz
````

### 2 setup local network
````bash
# enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1
# disable ICMP redirects
sudo sysctl -w net.ipv4.conf.all.send_redirects=0
````

### 3 add user mitmproxyuser
````bash
sudo useradd --create-home mitmproxyuser
sudo -u mitmproxyuser -H bash -c 'cd ~ && pip install --user mitmproxy'
````

### 4 add rules for redirecting local trafic to mitmproxy
````bash
sudo iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 443 -j REDIRECT --to-port 8080
sudo ip6tables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 80 -j REDIRECT --to-port 8080
sudo ip6tables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 443 -j REDIRECT --to-port 8080

# NOTICE: in case of removing these rules
sudo iptables -t nat -F
````

### 5 Certificates
By far the easiest way to install the mitmproxy CA certificate is to use the built-in certificate installation app. To do this, start mitmproxy and configure your target device with the correct proxy settings. Now start a browser on the device, and visit the magic domain mitm.it.

Download the certificate and install it
````bash
sudo cp mitmproxy.crt /usr/local/share/ca-certificates/mitmproxy.crt
sudo update-ca-certificates
````

### 6 launch mitmproxy
````bash
# mitmdump has console out 
# mitmproxy has an interactive console (shell)
# mitmweb launch a web version
sudo -u mitmproxyuser -H bash -c '$HOME/.local/bin/mitmdump --mode transparent --showhost --set block_global=false'
````

### 7 stop & clean all
````bash
# stop mitmproxy

# list rules
sudo iptables -L
# remove all
sudo iptables -X
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
````

## Example - terraform init proxies requests
After doing the setup above, I did a terraform init on a project, and wanted to check the requests. 

````bash
192.168.1.54:34112: client connect
192.168.1.54:34112: server connect 199.232.82.49:443
192.168.1.54:34112: GET https://registry.terraform.io/.well-known/terraform.json
                 << 200 OK 65b
192.168.1.54:34116: client connect
192.168.1.54:34116: server connect 199.232.82.49:443
192.168.1.54:34116: GET https://registry.terraform.io/v1/providers/kreuzwerker/docker/versions
                 << 200 OK 263b
192.168.1.54:34120: client connect
192.168.1.54:34120: server connect 199.232.82.49:443
192.168.1.54:34120: GET https://registry.terraform.io/v1/providers/kreuzwerker/docker/2.14.0/download/linux/amd64
                 << 200 OK 2.73k
192.168.1.54:44582: client connect
192.168.1.54:44582: server connect 140.82.121.4:443
192.168.1.54:44582: GET https://github.com/kreuzwerker/terraform-provider-docker/releases/download/v2.14.0/terraform-provider-docker_2.14.0_SHA256SUMS
                 << 302 Found 651b
192.168.1.54:55984: client connect
192.168.1.54:55984: server connect 185.199.108.154:443
192.168.1.54:55984: GET https://github-releases.githubusercontent.com/310596153/80c4d980-e097-11eb-95ad-a506af9ec4ca?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20210921%2Fus-east-1…
                 << 200 OK 1.46k
192.168.1.54:44582: GET https://github.com/kreuzwerker/terraform-provider-docker/releases/download/v2.14.0/terraform-provider-docker_2.14.0_SHA256SUMS.sig
                 << 302 Found 655b
192.168.1.54:55984: GET https://github-releases.githubusercontent.com/310596153/7efb1600-e097-11eb-928e-1dd31bfbc0af?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20210921%2Fus-east-1…
                 << 200 OK 566b
192.168.1.54:44590: client connect
192.168.1.54:44590: server connect 140.82.121.4:443
192.168.1.54:44590: GET https://github.com/kreuzwerker/terraform-provider-docker/releases/download/v2.14.0/terraform-provider-docker_2.14.0_linux_amd64.zip
                 << 302 Found 656b
192.168.1.54:47312: client connect
192.168.1.54:47312: server connect 185.199.109.154:443
192.168.1.54:47312: GET https://github-releases.githubusercontent.com/310596153/80c4d980-e097-11eb-8b9c-8b7dd255130d?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20210921%2Fus-east-1…
                 << 200 OK 6.97m

````

# mitmproxy

[![Continuous Integration Status](https://github.com/mitmproxy/mitmproxy/workflows/CI/badge.svg?branch=main)](https://github.com/mitmproxy/mitmproxy/actions?query=branch%3Amain)
[![Coverage Status](https://shields.mitmproxy.org/codecov/c/github/mitmproxy/mitmproxy/main.svg?label=codecov)](https://codecov.io/gh/mitmproxy/mitmproxy)
[![Latest Version](https://shields.mitmproxy.org/pypi/v/mitmproxy.svg)](https://pypi.python.org/pypi/mitmproxy)
[![Supported Python versions](https://shields.mitmproxy.org/pypi/pyversions/mitmproxy.svg)](https://pypi.python.org/pypi/mitmproxy)

``mitmproxy`` is an interactive, SSL/TLS-capable intercepting proxy with a console
interface for HTTP/1, HTTP/2, and WebSockets.

``mitmdump`` is the command-line version of mitmproxy. Think tcpdump for HTTP.

``mitmweb`` is a web-based interface for mitmproxy.

## Installation

The installation instructions are [here](https://docs.mitmproxy.org/stable/overview-installation).
If you want to install from source, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Documentation & Help

General information, tutorials, and precompiled binaries can be found on the mitmproxy website.

[![mitmproxy.org](https://shields.mitmproxy.org/badge/https%3A%2F%2F-mitmproxy.org-blue.svg)](https://mitmproxy.org/)

The documentation for mitmproxy is available on our website:

[![mitmproxy documentation stable](https://shields.mitmproxy.org/badge/docs-stable-brightgreen.svg)](https://docs.mitmproxy.org/stable/)
[![mitmproxy documentation dev](https://shields.mitmproxy.org/badge/docs-dev-brightgreen.svg)](https://docs.mitmproxy.org/main/)

If you have questions on how to use mitmproxy, please
ask them on StackOverflow!

[![StackOverflow: mitmproxy](https://shields.mitmproxy.org/stackexchange/stackoverflow/t/mitmproxy?color=orange&label=stackoverflow%20questions)](https://stackoverflow.com/questions/tagged/mitmproxy)

## Contributing

As an open source project, mitmproxy welcomes contributions of all forms.

[![Dev Guide](https://shields.mitmproxy.org/badge/dev_docs-CONTRIBUTING.md-blue)](./CONTRIBUTING.md)

Also, please feel free to join our developer Slack!

[![Slack Developer Chat](https://shields.mitmproxy.org/badge/slack-mitmproxy-E01563.svg)](http://slack.mitmproxy.org/)
````