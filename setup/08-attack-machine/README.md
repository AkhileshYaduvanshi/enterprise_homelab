## Overview

The Attack Machine represents an external adversary connected to the same physical network as the lab, but logically isolated by pfSense firewall rules.

## Design Goal

* Simulate real-world attacker from external network
* Allow controlled access to lab subnet (172.16.1.0/24)
* Log all malicious traffic through pfSense
* Maintain separation from internal lab systems


### Networks

Home Network (External)

```
192.168.1.0/24
```

Lab Network (Internal)

```
172.16.1.0/24
```

Attack Machine (UTM Kali)

```
Connected to 192.168.1.0/24
```

Important:

Attack machine is NOT inside lab network.

It must attack THROUGH pfSense.
