# Linux Networking Troubleshooting: Shop Site Recovery

## Overview

This project demonstrates hands-on troubleshooting of an unreachable internal web application on a Linux machine. The goal was to restore the shop website and verify connectivity layer by layer, following a practical DevOps troubleshooting workflow.

## Problem Statement

The following command must return the shop page over plain HTTP on port 8080:

```bash
curl http://shop.internal:8080/
```

Expected response:

```text
Welcome to the shop
```

## Intended Architecture

```text
Client
  |
  | http://shop.internal:8080/
  v
/etc/hosts
shop.internal -> 127.0.0.1
  |
  v
Nginx :8080
  |
  | Reverse proxy
  v
Python http.server :9000
  |
  v
/opt/shop/www
```

## Faults Identified

- Python backend was not listening on the required port 9000.
- An iptables INPUT rule dropped TCP traffic to port 8080.
- `shop.internal` resolved to an incorrect IP address instead of `127.0.0.1`.
- Nginx was inactive and not listening on port 8080.

## Troubleshooting and Resolution

### 1. Verify network interfaces and routing

```bash
ip link
ip addr
ip route
ip route show table local
```

Verified that the loopback interface was up, `127.0.0.1` was assigned, and local and default routes existed.

### 2. Start the Python backend

```bash
python3 -m http.server 9000 \
  --bind 127.0.0.1 \
  --directory /opt/shop/www \
  > /tmp/shop-backend.log 2>&1 &
```

Verify:

```bash
ss -tlnp | grep ':9000'
curl http://127.0.0.1:9000/
```

Expected response: `Welcome to the shop`.

### 3. Remove the blocking firewall rule

Inspect firewall rules:

```bash
iptables -L -n
```

Remove the specific rule that blocks TCP port 8080:

```bash
iptables -D INPUT -p tcp --dport 8080 -j DROP
```

Verify the rules again:

```bash
iptables -L -n
```

### 4. Correct hostname resolution

Inspect `/etc/hosts`:

```bash
grep -n 'shop.internal' /etc/hosts
```

Ensure the entry is:

```text
127.0.0.1 shop.internal
```

Verify:

```bash
getent hosts shop.internal
```

Expected IP address: `127.0.0.1`.

### 5. Validate and start Nginx

Check the configuration:

```bash
nginx -t
```

Start Nginx:

```bash
systemctl start nginx
```

Verify the listener:

```bash
ss -tlnp | grep ':8080'
```

The Nginx server block in `/etc/nginx/sites-enabled/shop` must listen on port 8080 and proxy requests to `http://127.0.0.1:9000`.

### 6. Verify end-to-end connectivity

```bash
curl -v http://shop.internal:8080/
```

Expected response:

```text
Welcome to the shop
```

## Skills Demonstrated

- Linux interface and IP troubleshooting
- Routing table inspection
- TCP listening-port diagnostics
- iptables firewall troubleshooting
- Hostname resolution using `/etc/hosts`
- Python HTTP server management
- Nginx reverse-proxy configuration
- Layer-by-layer service outage diagnosis
- End-to-end HTTP validation

## Key Learning

A service outage can involve multiple independent faults. Testing each layer separately—from the network interface through routing, listening ports, firewall rules, name resolution, and application response—helps isolate root causes and restore service reliably.

## Environment

- Linux
- Python `http.server`
- Nginx
- iptables
- curl
