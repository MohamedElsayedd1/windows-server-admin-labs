# Lab: Site-to-Site VPN Using RRAS Demand-Dial Interfaces (IKEv2)

## Overview

This lab documents how to connect **two private networks (branch offices)** together over the internet using **Windows Server Routing and Remote Access (RRAS)** with a **demand-dial VPN interface**. The tunnel uses **IKEv2** with **pre-shared key authentication**, and static routes are configured so each site's LAN subnet is reachable from the other side.

**Lab environment:**
- **Site 1 (Server: `PDC16`)** — LAN subnet `192.168.1.0/24`
- **Site 2** — LAN subnet `192.168.2.0/24`, reachable at `11.0.0.3` (public/routable address)
- VPN Type: **IKEv2** demand-dial (router-to-router / site-to-site)
- Authentication: **Pre-shared key**

**Goal:** Establish a persistent, always-on VPN tunnel between two RRAS routers so that hosts on each site's LAN can reach hosts on the other site's LAN transparently.

---

## Table of Contents

1. [Task 1 – Choose "Secure Connection Between Two Private Networks"](#task-1--choose-secure-connection-between-two-private-networks)
2. [Task 2 – Alternative: Custom Configuration Method](#task-2--alternative-custom-configuration-method)
3. [Task 3 – Assign IPv4 Address Ranges on Both Sites](#task-3--assign-ipv4-address-ranges-on-both-sites)
4. [Task 4 – Create the Demand-Dial Interface](#task-4--create-the-demand-dial-interface)
5. [Task 5 – Set the Connection as Persistent](#task-5--set-the-connection-as-persistent)
6. [Task 6 – Configure Pre-Shared Key Authentication on Both Sites](#task-6--configure-pre-shared-key-authentication-on-both-sites)
7. [Task 7 – Verify the VPN Tunnel Is Established](#task-7--verify-the-vpn-tunnel-is-established)
8. [Task 8 – Verify Both Sites Can Reach Each Other](#task-8--verify-both-sites-can-reach-each-other)
9. [Summary / Key Takeaways](#summary--key-takeaways)

---

## Task 1 – Choose "Secure Connection Between Two Private Networks"

Open **Routing and Remote Access** → right-click the server → **Configure and Enable Routing and Remote Access** to launch the wizard. On the **Configuration** page, select:

> **Secure connection between two private networks** — *"Connect this network to a remote network, such as a branch office."*

![RRAS Setup - Secure Connection](task01-rras-config-secure-connection.png)

**Why:** This is the purpose-built RRAS wizard option for site-to-site (router-to-router) VPNs. It automatically configures the server to accept and initiate demand-dial connections rather than end-user remote access.

---

## Task 2 – Alternative: Custom Configuration Method

Instead of the wizard's guided option, you can alternatively pick **Custom configuration** to manually select exactly which RRAS services to enable:

![RRAS Setup - Custom Configuration](task02-rras-config-custom.png)

On the **Custom Configuration** page, check:
- ☑ **VPN access**
- ☑ **Demand-dial connections (used for branch office routing)**

(Leave NAT and LAN routing unchecked unless also needed.)

![Custom Configuration - Select Services](task02b-custom-config-services.png)

**Why:** Custom configuration gives more granular control — useful if the server also needs other roles later, or if you want to enable demand-dial routing without the wizard forcing other unrelated services on.

---

## Task 3 – Assign IPv4 Address Ranges on Both Sites

Open the RRAS server's **Properties → IPv4** tab and confirm **Enable IPv4 Forwarding** is checked. If this server hands out addresses to VPN clients, define a static address pool, e.g.:

- **Start IP address:** `192.168.1.111`
- **End IP address:** `192.168.1.130`
- **Number of addresses:** `20`

![IPv4 Address Range](task03-ipv4-address-range.png)

**Why:** **IPv4 Forwarding** must be enabled for the server to route traffic *between* the two connected networks (not just terminate VPN sessions) — this is what makes it function as a router, not just a remote-access endpoint. This same step is performed on **both** sites, each using its own local subnet.

---

## Task 4 – Create the Demand-Dial Interface

In **Routing and Remote Access → Network Interfaces**, right-click → **New Demand-dial Interface** to launch the **Demand-Dial Interface Wizard**. The wizard's key steps:

### 4a. Interface Name
Give the interface a friendly name — typically named after the remote site/router it connects to, e.g. `site2`.

![Interface Name](task04h-interface-name.png)

### 4b. Connection Type
Select **Connect using virtual private networking (VPN)** (as opposed to a modem or PPPoE device).

![Connection Type - VPN](task04e-connection-type-vpn.png)

### 4c. VPN Type
Select **IKEv2**.

![VPN Type - IKEv2](task04d-vpn-type-ikev2.png)

**Why IKEv2:** It supports strong pre-shared key or certificate-based authentication, handles connection re-establishment well (important for a persistent site-to-site link), and is well suited for router-to-router tunnels.

### 4d. Destination Address
Enter the remote router's reachable host name or IP address — in this lab, Site 1 dials out to Site 2 at:

```
11.0.0.3
```

![Destination Address](task04b-destination-address.png)

### 4e. Protocols and Security
Check:
- ☑ **Route IP packets on this interface**
- ☑ **Add a user account so a remote router can dial in**

![Protocols and Security](task04f-protocols-and-security.png)

**Why:** "Route IP packets" enables this interface to forward traffic for the remote subnet. "Add a user account so a remote router can dial in" creates the local account the *other* site will use as its dial-out credentials when initiating the connection in the opposite direction.

### 4f. Dial-In Credentials
Set the username/password that the **remote** router will use when it dials **into** this server:

- **User name:** `site2`

![Dial-In Credentials](task04a-dial-in-credentials.png)

### 4g. Dial-Out Credentials
Set the username/password this interface will use when dialing **out to** the remote router. These must exactly match the dial-in credentials configured on the *remote* side:

- **User name:** `site1`

![Dial-Out Credentials](task04c-dial-out-credentials.png)

**Why both credential sets:** In a router-to-router VPN, authentication happens in both directions — the account matters because either side may be the one to initiate ("dial") the connection.

### 4h. Static Route
Add a static route so this router knows how to reach the *remote* site's LAN subnet through this demand-dial interface:

- **Destination:** `192.168.2.0`
- **Network Mask:** `255.255.255.0`
- **Metric:** `10`

![Static Route](task04g-static-route.png)

> ⚠️ **Note:** the screenshot shows a mask of `255.225.255.0`, which is a typo — the correct subnet mask for a standard `/24` network is **`255.255.255.0`**. Double check this field carefully when configuring your own routers, as a mistyped mask will silently break routing to the remote subnet.

**Why:** Without this static route, the server has no way of knowing that `192.168.2.0/24` traffic should be sent across the demand-dial tunnel — it would otherwise try to route it out the default gateway instead.

---

## Task 5 – Set the Connection as Persistent

Open the demand-dial interface's **Properties → Options** tab and set:

- **Connection type:** **Persistent connection** (instead of *Demand-dial*)

![Persistent Connection](task05-persistent-connection.png)

**Why:** By default, a demand-dial interface connects only when there's traffic to send and disconnects after an idle timeout. For a site-to-site link that should always be available (e.g., for site-to-site application traffic, monitoring, or low-latency access), setting it to **Persistent** keeps the tunnel up permanently and automatically redials if it drops.

---

## Task 6 – Configure Pre-Shared Key Authentication on Both Sites

Open the interface's **Properties → Security** tab:

- **Type of VPN:** `IKEv2`
- **Data encryption:** `Require encryption (disconnect if server declines)`
- **Authentication:** select **Use preshared key for authentication**
- **Key:** enter the same shared secret configured identically on **both** sites, e.g. `12341234`

![Pre-shared Key Authentication](task06-preshared-key-auth.png)

**Why:** IKEv2 needs a shared authentication method between both endpoints. A pre-shared key is the simplest option for a lab/small deployment (versus deploying machine certificates on each router). **This exact key must match on both routers**, or the IKE security association will fail to establish.

---

## Task 7 – Verify the VPN Tunnel Is Established

Back in the **Network Interfaces** view of Routing and Remote Access, confirm the demand-dial interface shows:

| Interface | Connection Type | Status | Connection State |
|---|---|---|---|
| `site2` | Demand-dial | Enabled | **Connected** |

![VPN Established](task07-vpn-established.png)

**Why:** This confirms the IKEv2 tunnel and PPP negotiation completed successfully and the interface is actively passing traffic (not just configured, but genuinely up).

---

## Task 8 – Verify Both Sites Can Reach Each Other

From Site 1, ping a host on Site 2's LAN subnet to confirm end-to-end reachability across the tunnel:

```
C:\Users\Administrator>ping 192.168.2.2

Pinging 192.168.2.2 with 32 bytes of data:
Reply from 192.168.2.2: bytes=32 time=1ms TTL=127
Reply from 192.168.2.2: bytes=32 time<1ms TTL=127
Reply from 192.168.2.2: bytes=32 time=1ms TTL=127
Reply from 192.168.2.2: bytes=32 time=1ms TTL=127

Ping statistics for 192.168.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

![Sites Can Reach Each Other](task08-sites-reach-each-other.png)

**Result:** 0% packet loss confirms the site-to-site VPN tunnel is fully operational, routing between `192.168.1.0/24` and `192.168.2.0/24` is correctly configured, and traffic is passing through the IKEv2 demand-dial interface as expected.

---

## Summary / Key Takeaways

| Step | Purpose |
|---|---|
| Secure connection between two private networks (or Custom + VPN access + Demand-dial connections) | Enables RRAS's site-to-site routing role |
| Enable IPv4 Forwarding | Allows the server to route packets between networks, not just terminate sessions |
| Demand-Dial Interface (IKEv2) | Defines how/where this router dials out to reach the remote router |
| Dial-in vs. Dial-out credentials | Authenticates each direction of the connection separately; must match on both ends |
| Static route to remote subnet | Tells the router which traffic belongs on the demand-dial interface |
| Persistent connection | Keeps the tunnel always-on rather than only connecting on-demand |
| Pre-shared key (IKEv2) | Provides mutual authentication between the two routers without needing PKI/certificates |

**Configuration must be mirrored on both routers** — matching VPN type (IKEv2), matching pre-shared key, matching (opposite) dial-in/dial-out credentials, and a static route pointing to the *other* site's subnet. A mismatch in any of these (as illustrated by the subnet mask typo in Task 4h) is one of the most common causes of a tunnel that connects but doesn't actually pass traffic.
