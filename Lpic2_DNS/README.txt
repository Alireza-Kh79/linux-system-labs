# DNS Lab Scenario – BIND9 on Ubuntu (Educational)

## Purpose of This Scenario
This scenario is designed strictly for **educational and learning purposes**.

The goal of this lab is to demonstrate:
- How DNS works internally
- How an authoritative DNS server is configured
- How DNS zones and records are defined using BIND9
- How to validate and troubleshoot DNS configurations

## Security Note  
In real production environments, it is **not recommended** to run a DNS server and a web server on the same machine.
They should be separated to:
- Reduce attack surface
- Improve security
- Increase stability and scalability

In this lab, both services may coexist **only for simplicity and learning**.

---

## Lab Topology

DNS Server:
- OS: Ubuntu
- Service: BIND9
- IP: 192.168.100.106

Client:
- OS: Fedora

Network:
- 192.168.100.0/24

Client  --->  DNS Server


---

## Relation to Previous Scenario

This DNS lab is built on top of a previous web server scenario.

In the earlier scenario:
- An Ubuntu server was configured with Nginx
- Virtual hosts were defined for `site1` and `site2`
- Each site was served from its own document root

In this lab:
- DNS records (`site1.lab.local`, `site2.lab.local`) resolve to the same server
- DNS is used to map hostnames to the existing web services

You can find the web server setup in the following repository:

👉 https://github.com/Alireza-Kh79/linux-system-labs/tree/main/NGINX%20Name%20Based%20Virtual%20Hosting
---

## Step 1 – Install BIND9 on Ubuntu

Update package lists and install required packages:

```bash
sudo apt update
sudo apt install bind9 bind9utils dnsutils
```

Verify service status:

```bash
systemctl status bind9
```

---

## Step 2 – Define DNS Zone

Edit the local BIND configuration file:

File:
```
/etc/bind/named.conf.local
```

Add the following zone definition:

```conf
zone "lab.local" {
    type master;
    file "/etc/bind/zones/db.lab.local";
};
```

This means:
- This server is the authoritative (master) DNS server for `lab.local`
- Zone data is stored in the specified file

---

## Step 3 – Create Zone File

Create directory and zone file:

```bash
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.lab.local
```

Zone file content:

```dns
$TTL 604800
@   IN  SOA ns1.lab.local. admin.lab.local. (
        2024020501 ; Serial
        604800     ; Refresh
        86400      ; Retry
        2419200    ; Expire
        604800 )   ; Negative Cache TTL

@       IN  NS      ns1.lab.local.
ns1     IN  A       192.168.100.106
site1   IN  A       192.168.100.106
site2   IN  A       192.168.100.106
```

---

## Zone File Syntax Explanation

- `$TTL`  
  Sets the default Time To Live for records in this zone.

- `@`  
  Represents the current zone name (`lab.local`).

- `SOA (Start of Authority)`  
  Contains administrative and timing information for the zone.

- `NS Record`  
  Defines the authoritative name server for the zone.

- `ns1`  
  Hostname of the DNS server itself.

- `A Record`  
  Maps a hostname to an IPv4 address.

---

## Step 4 – Validate Configuration

Check global BIND configuration:

```bash
sudo named-checkconf
```

Check zone file syntax:

```bash
sudo named-checkzone lab.local /etc/bind/zones/db.lab.local
```

Expected result:
```
OK
```

---

## Step 5 – Restart DNS Service

```bash
sudo systemctl restart bind9
```

---

## Step 6 – Test DNS Resolution

From the DNS server:

```bash
dig @127.0.0.1 site1.lab.local
```

From the client (Fedora):

```bash
dig @192.168.100.106 site1.lab.local
```

Expected result:
- `status: NOERROR`
- Correct IP address returned

---

## Screenshots to Include in README

1. named.conf.local zone definition
2. db.lab.local zone file
3. Successful named-checkzone output
4. bind9 service status
5. dig command showing NOERROR

---

## Final Notes

DNS is a large and specialized field.
For LPIC-2 and system administration roles:
- Strong fundamentals are sufficient
- Deep specialization can be learned when required

This lab demonstrates practical, real-world DNS concepts at the LPIC-2 level.