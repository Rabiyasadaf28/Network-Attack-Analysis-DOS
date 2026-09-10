# Network Attack Analysis — SYN Flood DoS

**One line:** Investigated a web server outage using packet capture, identified a
TCP SYN flood denial-of-service attack from a single source IP, and documented the
resource-exhaustion mechanism, business impact, and mitigation options.

## Scenario

As a security analyst for a travel agency whose staff rely on the company
sales webpage to find vacation packages for customers.

One afternoon an automated monitoring alert flagged a problem with the web server.
Attempting to load the company website returned a **connection timeout** error in
the browser. Using a packet sniffer to capture traffic to and from the web server,
I observed a large volume of **TCP SYN requests originating from a single
unfamiliar IP address**. The server was being overwhelmed by the incoming volume
and losing its ability to respond.

Immediate actions taken: took the server offline temporarily to let it recover,
and configured the firewall to block the offending source IP. I recognised that IP
blocking is a short-term measure — an attacker can spoof other addresses to bypass
it — so the incident needed escalation to my manager for a durable fix.

📄 [Cybersecurity incident report .docx](https://github.com/user-attachments/files/32051197/Cybersecurity.incident.report.docx)


## Approach

1. **Captured traffic** with a packet sniffer to see the connection attempts at
   the packet level.
2. **Looked for patterns rather than single packets** — volume, source
   distribution, and which stage of the TCP handshake was failing.
3. **Identified the attack type** by matching the observed pattern against known
   network intrusion types.
4. **Contained** — server offline to recover, source IP blocked at the firewall.
5. **Documented and escalated**, including why containment was temporary.

## Log analysis

| Observation | Interpretation |
|---|---|
| High volume of TCP SYN packets | Connection requests, not data transfer |
| All from one unfamiliar source IP | Single-source — DoS, not DDoS |
| Server stops responding after the volume spike | Resource exhaustion, not a crash or misconfiguration |
| Legitimate visitors receive connection timeout | Server has no resources left to complete new handshakes |
| No completed three-way handshakes from the source | Half-open connections accumulating |

**Why a connection timeout and not an error page:** an HTTP error means the server
received the request and answered. A timeout means the connection was never
established — the failure sits at the TCP layer, below HTTP. That single
distinction is what points the investigation at the handshake.

## DoS vs DDoS

| | DoS | DDoS |
|---|---|---|
| Sources | Single attacking machine | Many distributed machines, usually a botnet |
| Blocking by IP | Effective short-term | Impractical — too many sources |
| Present in this incident | ✅ | ❌ |

This was a DoS. Blocking the single source IP worked precisely *because* it was
single-source — which is also why the fix is fragile: the same attacker can spoof
or rotate addresses.

## Completed incident report

### Section 1: Identify the type of attack that may have caused this network interruption

One potential explanation for the website's connection timeout error message is a
DoS attack. The logs show that the web server stops responding after it is
overloaded with SYN packet requests. This event could be a type of DoS attack
called SYN flooding.

### Section 2: Explain how the attack is causing the website malfunction

When the website visitors try to establish a connection with the web server, a
three-way handshake occurs using the TCP protocol. The handshake consists of three
steps: A SYN packet is sent from the source to the destination, requesting to
connect. The destination replies to the source with a SYN-ACK packet to accept the
connection request. The destination will reserve resources for the source to
connect. A final ACK packet is sent from the source to the destination
acknowledging the permission to connect.

In the case of a SYN flood attack, a malicious actor will send a large number of
SYN packets all at once, which overwhelms the server's available resources to
reserve for the connection. When this happens, there are no server resources left
for legitimate TCP connection requests. The logs indicate that the web server has
become overwhelmed and is unable to process the visitors' SYN requests. The server
is unable to open a new connection to new visitors who receive a connection
timeout message.

## Findings

| Item | Conclusion |
|---|---|
| Attack type | TCP SYN flood |
| Attack class | Denial of service (DoS) — single source |
| Protocol abused | TCP — three-way handshake |
| Mechanism | Half-open connections consume the server's connection backlog |
| Affected asset | Public-facing web server (sales webpage) |
| Symptom to users | Connection timeout |
| Containment applied | Server taken offline to recover; source IP blocked at firewall |
| Containment durability | Low — defeated by IP spoofing or source rotation |

**Business impact:** the sales webpage is how staff find packages for customers, so
the outage stops revenue-generating work, not just external browsing. Customers
hitting a dead site during a promotion causes reputational and conversion loss,
and the incident consumed analyst and engineering time. Extended or repeated
outages would raise questions about availability commitments to customers.

## Mitigation recommendations

| Control | Addresses |
|---|---|
| SYN cookies enabled on the web server | Removes the need to reserve resources for unverified half-open connections |
| Rate limiting / connection throttling at the firewall or load balancer | Caps requests per source regardless of address |
| Upstream DDoS protection service (e.g. CDN-fronted scrubbing) | Absorbs volume before it reaches origin, and survives source rotation |
| Reduced SYN-ACK retry timeout | Frees half-open slots faster |
| Baseline traffic monitoring with alerting on SYN-rate anomalies | Detects the next attempt earlier than a user-reported outage |
| Documented DoS response runbook | Removes improvisation from containment next time |

Ranked deliberately: SYN cookies and rate limiting are host-level and free,
whereas upstream scrubbing is the only one that holds against a genuine
distributed attack.

## Environment

Wireshark / packet capture · TCP three-way handshake · SYN flood · DoS vs DDoS ·
firewall IP blocking · incident report template

---
*Completed as a Google Cybersecurity Certificate activity; extended with a
DoS/DDoS distinction, business impact assessment, ranked mitigation options, and
an explicit limitations assessment.*
