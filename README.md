# A Highly Available, Load Balanced Smart Host Relay (to Mimecast)

# Summary
This setup provides a highly available on-premise (or cloud hosted) SMTP relay where mail can be received without TLS (based on this configuration,for legacy applications) and enforced TLS (1.3) over the internet to Mimecast smarthosts.

# Architecture
In this setup, we will use `keepalived` to provide High Availability (HA) between two Debian hosts running `haproxy` listening on TCP/25 and postfix listening on TCP/2525. `keepalived` uses VRRP, where each host will have their own IP on each interface, and a floating IP address will be shared between the two hosts in active/standby configuration. 

While `keepalived` on each host will float and IP in active/standby, the postfix MTA on each host will send mail in active/active configuration via `haproxy` which will monitor the health of each `postfix` instance via `option smtpchk HELO localhost` in the `haproxy` configuration.

`haproxy` is an open-source load balancer application that supports TCP load balancing. `haproxy` will run on both hosts listening on TCP/25 and load-balance traffic evenly to the two postfix MTA (regardless of which host is active from a `keepalived`/VRRP point of view).

`postfix` will be configured to accept unauthenticated SMTP relay in plain-text or TLS(opportunistic TLS) from specific LAN networks e.g. 192.168.0.0/23 without SASL authentication.

This specific configuration was tested on two debian 11 hosts.

# Step 1 - Install the software
The first thing to do is install `postfix` on your hosts - you can do this on both hosts or after install clone the VM. To install the required software run:

````
apt update
apt install -Y postfix keepalived haproxy curl nano
````

# Step 2 - Configure Postfix
There are two main files required for postfix. `master.cf` and `main.cf` where `master.cf` has information relating to the IP and port `postfix` will listen on, and `main.cf` has MTA specific configuration.

I am hosting a sample copy of each here with comments. 

These files should be uploaded to /etc/postfix/ folder

Full postfix documentation is available her: https://www.postfix.org/documentation.html

Enable the service and start.
````
systemctl enable postfix
systemctl restart postfix
````

You should now be able to telnet to localhost:25 and be served with a SMTP banner.

# Step 3 - Configure HAproxy
HAProxy is a powerful open-source load balancer. The configuration file is stored in `/etc/haproxy/haproxy.cfg`

A smaple copy is available in this repo. 

`haproxy.cfg` should be copied to /etc/haproxy/ folder

Enable the service and start.
````
systemctl enable haproxy
systemctl restart haproxy
````

# Step 4 - Configure Keepalived
Keepalived brings high-availability to linux.

Enable the service and start.
````
systemctl enable keepalived
systemctl restart keepalived

# Final step - Mimecast (if using Mimecast as smart host)

In order to relay outbound via Mimecast, you will need to ensure your WAN IP address that SMTP egresses from is added to the Authorized Outbound IP configuration. This can be a single IP or CIDR netblock.
