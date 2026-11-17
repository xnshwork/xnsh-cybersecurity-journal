
# Lab 05 — TCP Three-Way Handshake & HTTP Traffic Analysis

**Author:** xnsh  
**Date:** 17 November 2026  
**Category:** Networking & Traffic Analysis  
**Tools:** Kali Linux (WSL2), Wireshark, Linux Terminal  
**Status:** Completed

---

## 1. Objective

The objective of this lab was to capture and analyze a complete TCP connection using Wireshark.

The investigation focused on understanding how TCP establishes connections, how HTTP requests and responses travel through them, and how TCP connections terminate gracefully.

We also inspected TCP flags, ports, sequence numbers, acknowledgment numbers, and HTTP status codes.

## 2. Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux (WSL2) |
| Host System | Windows 11 |
| Packet Analyzer | Wireshark |
| Capture Interface | eth0 |
| Client IP | 172.29.9.207 |
| Server IP | 172.66.147.243 |
| Source Port | 34942 |
| Destination Port | 80 |
| Protocols | TCP, HTTP |
| Display Filter | `tcp` |

## 3. Concepts Covered

**TCP (Transmission Control Protocol)**

A transport-layer protocol providing reliable, ordered communication between applications.

**Three-Way Handshake**

TCP establishes a connection using three packets:

1. SYN — Client initiates the connection.
2. SYN-ACK — Server acknowledges and responds.
3. ACK — Client acknowledges the server, completing the handshake.

**TCP Ports**

Ports identify application endpoints. The client used temporary port `34942`, while the server received the connection on HTTP port `80`.

**Sequence and Acknowledgment Numbers**

Sequence numbers track positions in the TCP byte stream. Acknowledgment numbers indicate the next sequence number expected from the other endpoint.

**HTTP (Hypertext Transfer Protocol)**

An application-layer protocol used to exchange web requests and responses.

**FIN (Finish)**

A TCP flag indicating that a device has finished sending data and wants to close its sending side of the connection.

## 4. Methodology

### Step 1 — Capturing TCP Traffic

Started Wireshark on the `eth0` network interface.

Executed the following command in the Kali terminal:

```bash
curl -4 -v --http1.1 http://example.com/
```

Command explanation:

- `curl` — Sends a request to a web server.
- `-4` — Forces IPv4 communication.
- `-v` — Displays detailed connection information.
- `--http1.1` — Requests HTTP/1.1.
- `http://example.com/` — Target website.

After generating the traffic, stopped the capture and applied the display filter:

```text
tcp
```

The captured packets included the TCP handshake, HTTP request, HTTP response, and connection termination.

### Step 2 — Source and Destination Port Inspection

Selected packet No. 1, the initial SYN packet.

Observed:

| Field | Value |
|---|---|
| Source IP | 172.29.9.207 |
| Destination IP | 172.66.147.243 |
| Source Port | 34942 |
| Destination Port | 80 |

The source port was an ephemeral port selected by Kali for this connection.

Port `80` was the destination service port used for HTTP communication.

The server's response reversed the source and destination ports.

**Evidence: TCP Source and Destination Ports**

![TCP source and destination ports](05-tcp-source-destination-ports.png)

### Step 3 — TCP Three-Way Handshake

Inspected the first three packets.

| Packet | Flags | Hex Value | Direction |
|---|---|---|---|
| 1 | SYN | 0x02 | Kali → Server |
| 2 | SYN, ACK | 0x12 | Server → Kali |
| 3 | ACK | 0x10 | Kali → Server |

**Packet 1 — SYN**

Kali initiated the connection by sending a SYN packet to the server.

**Packet 2 — SYN-ACK**

The server acknowledged Kali's request and sent its own SYN.

Both the SYN and ACK flags were set.

**Evidence: TCP SYN-ACK Flags**

![TCP SYN and ACK flags inspection](05-tcp-syn-ack-flags.png)

**Packet 3 — ACK**

Kali acknowledged the server's SYN, completing the TCP three-way handshake.

Observed relative sequence and acknowledgment numbers:

- Sequence Number: `1`
- Acknowledgment Number: `1`
- ACK Flag: Set

The SYN packets each consumed one sequence number. Wireshark displayed relative sequence numbers to simplify analysis.

**Evidence: TCP Sequence and Acknowledgment Numbers**

![TCP sequence and acknowledgment number inspection](05-tcp-sequence-acknowledgment.png)

### Step 4 — HTTP GET Request Analysis

After establishing the TCP connection, Kali sent an HTTP request.

Selected packet No. 4.

Observed:

```http
GET / HTTP/1.1
Host: example.com
```

**Field explanations:**

- `GET` — Requests a resource from the server.
- `/` — Represents the root path of the website.
- `HTTP/1.1` — Specifies the HTTP version.
- `Host: example.com` — Identifies the requested website.

This demonstrated how HTTP application data is transmitted after TCP connection establishment.

**Evidence: HTTP GET Request**

![HTTP GET request and Host header](05-http-get-request.png)

### Step 5 — HTTP Response Analysis

Selected packet No. 7 to inspect the server's response.

Observed:

| Field | Value |
|---|---|
| Response Version | HTTP/1.1 |
| Status Code | 200 |
| Response Phrase | OK |
| Content Type | text/html |
| Server Header | cloudflare |

**HTTP 200 OK** indicates that the HTTP request was successfully handled.

This is different from TCP connection success. A TCP connection can be established even when an HTTP request returns an error.

Wireshark also showed that the HTTP response was reassembled from TCP segments.

**Evidence: Successful HTTP Response**

![HTTP 200 OK response analysis](05-http-200-ok-response.png)

### Step 6 — TCP Connection Termination

Finally, inspected packets No. 10, 11, and 12.

| Packet | Direction | Flags |
|---|---|---|
| 10 | Kali → Server | FIN, ACK |
| 11 | Server → Kali | FIN, ACK |
| 12 | Kali → Server | ACK |

**Packet 10**

Kali initiated graceful connection closure by sending FIN-ACK.

**Packet 11**

The server acknowledged the request and indicated that it had finished sending data.

**Packet 12**

Kali acknowledged the server's FIN, completing the observed closing exchange.

The FIN-ACK hexadecimal flag value was `0x11`.

Wireshark also identified packet No. 10 as initiating the connection closure.

**Evidence: TCP Connection Termination**

![TCP FIN ACK and connection termination](05-tcp-connection-termination.png)

## 5. Observations

The packet capture demonstrated the complete lifecycle of a TCP connection.

- The client initiated communication using a temporary source port.
- The TCP three-way handshake successfully established the connection.
- TCP sequence and acknowledgment numbers confirmed the handshake progression.
- An HTTP GET request was transmitted after connection establishment.
- The server returned HTTP `200 OK`, indicating a successful HTTP response.
- The connection closed gracefully using FIN and ACK packets.
- Wireshark allowed inspection of decoded protocol fields and hexadecimal packet data.

## 6. Troubleshooting and Notes

**HTTP packets appearing with the TCP filter**

Some packets were displayed as HTTP even though the `tcp` filter was applied. This happened because Wireshark recognized application-layer HTTP data carried over TCP.

**Sequence number differences**

Wireshark displayed relative sequence numbers rather than raw TCP sequence numbers, making the handshake easier to follow.

**Evidence collection**

Screenshots were captured for important observations rather than every individual packet, keeping the journal focused on meaningful findings.

## 7. Real-World Cybersecurity Applications

**Network Troubleshooting**

TCP flags and acknowledgment numbers help identify connection failures, retransmissions, and abnormal connection behavior.

**Port Scanning**

Understanding SYN and SYN-ACK packets provides a foundation for interpreting authorized Nmap TCP scan results.

**Web Application Security**

HTTP methods, headers, and response codes are essential when investigating application behavior using tools such as Burp Suite.

**Traffic Analysis**

Wireshark helps analysts reconstruct communication sequences and investigate suspicious or unexpected network activity.

## 8. Key Learnings

Through this lab, I learned:

1. How TCP establishes a connection using SYN, SYN-ACK, and ACK.
2. Why client and server port numbers are different.
3. How TCP sequence and acknowledgment numbers work during connection establishment.
4. How HTTP requests and responses travel over TCP.
5. How to interpret HTTP status codes such as `200 OK`.
6. How TCP gracefully closes connections using FIN and ACK.
7. How to inspect protocol flags and packet data using Wireshark.
8. How to document packet-level observations with reproducible evidence.

## 9. Conclusion

Successfully captured and analyzed a complete TCP connection using Kali Linux and Wireshark.

The investigation followed communication from the initial TCP SYN packet through the HTTP GET request, successful HTTP response, and graceful connection termination.

The lab strengthened my understanding of transport-layer communication and provided practical experience relevant to network troubleshooting, traffic analysis, and web application security.

**Lab Status:** Completed  
**Evidence Collected:** 6 screenshots  
**Next Focus:** TCP retransmissions, resets, and connection troubleshooting.
