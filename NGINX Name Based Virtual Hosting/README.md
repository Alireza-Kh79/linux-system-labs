# NGINX Name-Based Virtual Hosting (LPIC-2 Practical Lab)

This document describes a practical LPIC-2 level lab focused on configuring
and understanding name-based virtual hosting using NGINX on a Linux system.

The purpose of this lab is not only to serve web pages, but to deeply understand
how NGINX selects server blocks based on the Host header, how default servers
work, and how site isolation is implemented using sites-available and
sites-enabled.

---

## Environment

- OS: Ubuntu (systemd-based)
- Web Server: NGINX
- Network: Localhost (127.0.0.1)
- Tools: curl, ss, systemctl

---

## 1. Problem Definition

The goal of this scenario was to configure multiple websites on a single IP
address using NGINX name-based virtual hosting.

Objectives:

- Install and manage NGINX as a systemd service
- Configure multiple server blocks on the same IP and port
- Route requests based on the Host header
- Understand default server behavior
- Validate configuration using command-line tools
- Document and verify expected behavior

---

## 2. NGINX Installation and Service Management

NGINX was installed using the system package manager:

sudo apt update  
sudo apt install nginx  

The service was managed using systemd:

systemctl enable nginx  
systemctl start nginx  
systemctl status nginx  

Listening sockets were verified:

ss -tulnp | grep :80  

---

## 3. Server Block Structure

Two separate server blocks were created to simulate two different websites
hosted on the same IP address.

Each server block was defined as a separate configuration file under:

/etc/nginx/sites-available/

Example files:

- site1.conf
- site2.conf

Each server block contained:

- listen directive on port 80
- server_name directive
- root directory for content
- index directive
- basic location block

---

## 4. Name-Based Virtual Hosting

Both server blocks were configured to listen on:

127.0.0.1:80

Routing was based purely on the Host header provided by the client.

Example server_name values:

- site1.local
- site2.local

This demonstrates name-based virtual hosting, where multiple websites share
the same IP address and port.

---

## 5. sites-available and sites-enabled Mechanism

The server block configuration files were placed in:

/etc/nginx/sites-available/

They were then enabled using symbolic links:

ln -s /etc/nginx/sites-available/site1.conf /etc/nginx/sites-enabled/  
ln -s /etc/nginx/sites-available/site2.conf /etc/nginx/sites-enabled/  

This approach allows:

- Clean separation of available vs active configurations
- Safe enabling/disabling of sites
- Easier administration and troubleshooting

---

## 6. Default Server Behavior

During testing, it was observed that when a request does not match any
server_name, NGINX serves the first loaded server block on that port.

This behavior was resolved by removing the default configuration and ensuring
only intended server blocks were enabled.

This demonstrates how NGINX selects a fallback server when no match is found.

---

## 7. Configuration Testing and Reloading

Before applying changes, configuration syntax was validated:

nginx -t  

After successful validation, NGINX was reloaded:

systemctl reload nginx  

This ensured zero downtime while applying configuration changes.

---

## 8. Verification Using curl

Requests were tested using curl to explicitly control the Host header.

Examples:

curl -H "Host: site1.local" http://127.0.0.1  
curl -H "Host: site2.local" http://127.0.0.1  



---

## 9. Key Learnings

- NGINX does not route traffic based on IP address alone
- The Host header is critical for name-based virtual hosting
- Unmatched requests are handled by the default server
- sites-available and sites-enabled provide safe configuration management
- curl is an essential tool for debugging HTTP behavior
- Proper service management with systemd is required in production systems

---

## 10. Conclusion

This lab demonstrates a core LPIC-2 skill: managing network services with
predictable, secure, and well-documented behavior.

The scenario reflects real-world Linux system administration tasks involving
web servers, networking, and service management.