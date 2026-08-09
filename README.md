# Wireshark Traffic Dissection and File Extraction Lab

This guide covers how to analyze a saved packet capture, isolate HTTP and FTP traffic, follow TCP streams, extract transferred files, and perform a live capture from an HTB Academy lab system.

> **Investigation scope:** Examine traffic involving `172.16.10.2` and `172.16.10.20`, identify suspicious transfers, and recover relevant files.

## Commands Covered

| Command, option, or tool  | Purpose                                                         |
| ------------------------- | --------------------------------------------------------------- |
| **Wireshark**             | Captures, filters, inspects, and reconstructs network traffic.  |
| **Wireshark-lab-2.zip**   | Optional lab resource containing the provided capture.          |
| **Wireshark-lab-2.pcap**  | Saved packet capture containing the unencrypted web session.    |
| **File → Open**           | Opens an existing `.pcap` file in Wireshark.                    |
| `http`                    | Displays packets Wireshark identifies as HTTP.                  |
| **TCP port 80**           | Standard port used by the HTTP traffic in this lab.             |
| **GET**                   | HTTP request method used to retrieve resources.                 |
| **200 OK**                | HTTP response indicating that a request succeeded.              |
| **Follow → TCP Stream**   | Reassembles and displays one TCP conversation.                  |
| **JFIF**                  | File signature commonly found inside JPEG images.               |
| **PNG**                   | Another common image format that may appear in HTTP transfers.  |
| **Export Objects → HTTP** | Extracts files transferred through HTTP.                        |
| **XFreeRDP / FreeRDP**    | Provides remote desktop access to the lab machine.              |
| `xfreerdp`                | Starts a FreeRDP connection from the terminal.                  |
| `/v:`                     | Specifies the remote desktop server address.                    |
| `/u:`                     | Specifies the RDP username.                                     |
| `/p:`                     | Specifies the RDP password.                                     |
| **HTB Academy VPN**       | Connects a local VM to the Academy lab network.                 |
| **Pwnbox**                | Browser-accessible HTB attack system that can run FreeRDP.      |
| **ENS224**                | Network interface used for the live Wireshark capture.          |
| **Conversations**         | Wireshark view that summarizes communication between endpoints. |
| `ftp`                     | Displays FTP control-channel commands and responses.            |
| `ftp-data`                | Displays FTP file-transfer traffic.                             |
| **FTP commands**          | Reveal authentication, filenames, and transfer actions.         |

---

## 1. Open the Provided Packet Capture

### Purpose

Load the previously recorded network session into Wireshark.

### Procedure

Extract `Wireshark-lab-2.zip`, start Wireshark, and use:

```text
File → Open
```

Select:

```text
Wireshark-lab-2.pcap
```

### Example

```text
File → Open → Wireshark-lab-2.pcap
```

The capture contains approximately 1,171 packets, but fewer than 20 are specifically identified as HTTP.

---

## 2. Filter for HTTP Traffic

### Purpose

Remove unrelated IP and TCP packets so that the web session is easier to examine.

### Filter syntax

```wireshark
http
```

### Example

Enter the following in Wireshark’s display-filter bar:

```wireshark
http
```

Press Enter or apply the filter.

### Expected result

The packet list should primarily show HTTP messages such as:

```text
GET /requested-file HTTP/1.1
HTTP/1.1 200 OK
```

A `GET` packet represents a client requesting a resource. A `200 OK` response indicates that the server successfully returned it.

> The `http` display filter shows both requests and responses, allowing the request and its returned object to be examined together.

---

## 3. Follow an HTTP TCP Stream

### Purpose

Reassemble the individual TCP packets belonging to one HTTP transaction.

### Procedure syntax

```text
Select a 200 OK packet
→ Right-click
→ Follow
→ TCP Stream
```

### Example

```text
HTTP/1.1 200 OK
→ Follow
→ TCP Stream
```

The stream window combines the client request and server response into one readable conversation.

### What to inspect

Look for:

```text
GET
200 OK
Content-Type
Content-Length
JFIF
PNG
```

The stream may contain readable HTTP headers followed by binary file data.

---

## 4. Identify JPEG or PNG Content

### Purpose

Confirm whether the HTTP response contains an image before exporting it.

### JFIF indicator

JPEG files commonly contain the following identifier:

```text
JFIF
```

### Example

Inspect the packet details, packet bytes, or followed TCP stream for:

```text
JFIF
```

The presence of `JFIF` strongly suggests that the transferred object is a JPEG image.

PNG files may instead be identified by their format information or content type:

```text
image/png
```

### Important context

The security manager suspects that data was concealed inside an image. Extracting the original file preserves it for further steganography or forensic analysis.

---

## 5. Export HTTP Objects

### Purpose

Recover complete files transferred through the captured HTTP session.

### Procedure syntax

```text
File
→ Export Objects
→ HTTP
```

### Example

```text
File → Export Objects → HTTP
```

In the HTTP Objects window:

1. Review the listed filenames and content types.
    
2. Select the suspected image.
    
3. Choose **Save** or **Save All**.
    
4. Store the evidence in an investigation directory.
    

### Result

Wireshark reconstructs the selected object from its packets and saves it as a normal file.

Do not modify the recovered file before forensic examination.

---

## 6. Connect to the Live Lab with FreeRDP

### Purpose

Open the lab machine’s graphical desktop so Wireshark can capture traffic from inside the environment.

### Command syntax

```bash
xfreerdp /v:<TARGET_IP> /u:htb-student /p:'HTB_@cademy_stdnt!'
```

### Example

Replace `<TARGET_IP>` with the address shown after spawning the target:

```bash
xfreerdp /v:<TARGET_IP> /u:htb-student /p:'HTB_@cademy_stdnt!'
```

### Options

|Option|Meaning|
|---|---|
|`/v:<TARGET_IP>`|Remote system running the desktop service|
|`/u:htb-student`|Lab username|
|`/p:'HTB_@cademy_stdnt!'`|Lab password|

The password is quoted because the exclamation mark can have special meaning in some shells.

The connection can be launched from either:

```text
Your own VM connected through the HTB Academy VPN
```

or:

```text
The Pwnbox provided in the module
```

---

## 7. Start a Live Wireshark Capture

### Purpose

Observe current communications involving the suspicious host.

### Capture procedure syntax

```text
Wireshark
→ Select ENS224
→ Start Capture
```

### Example

```text
Capture interface: ENS224
```

Allow the capture to run for several minutes, then stop it:

```text
Capture → Stop
```

Focus the investigation on:

```text
172.16.10.2
172.16.10.20
```

Save the capture if it may be needed as evidence or for later analysis.

---

## 8. Analyze Hosts and Conversations

### Purpose

Determine which systems are communicating and what roles they appear to have.

### Procedure syntax

```text
Statistics → Conversations
```

### Example

```text
Statistics → Conversations → IPv4
```

Review:

- The number of conversations.
    
- Source and destination addresses.
    
- Packet and byte counts.
    
- Frequently used protocols and ports.
    
- Which host initiates each connection.
    

Ask the following questions:

- Which systems appear to be clients?
    
- Which systems appear to be servers?
    
- What protocols are being used?
    
- Are unexpected ports carrying recognizable protocols?
    
- Are credentials or other sensitive data visible in clear text?
    

---

## 9. Analyze FTP Control Traffic

### Purpose

Inspect FTP commands, authentication, filenames, and transfer activity.

### Filter syntax

```wireshark
ftp
```

### Example

```wireshark
ftp
```

FTP control traffic may expose commands and responses such as:

```text
USER
PASS
RETR
STOR
```

Use the visible commands to determine:

- Which host is the FTP server.
    
- Whether a named user or anonymous account authenticated.
    
- Whether a file was uploaded or downloaded.
    
- The name of the transferred file.
    

Because traditional FTP is unencrypted, usernames and passwords may be visible directly in the capture.

---

## 10. Inspect and Reassemble FTP Data

### Purpose

Locate the packets carrying the actual transferred file rather than the FTP commands.

### Filter syntax

```wireshark
ftp-data
```

### Example

```wireshark
ftp-data
```

The `ftp` filter shows the control conversation, while `ftp-data` shows the file contents transferred through the separate data connection.

### Extraction workflow

1. Use `ftp` to identify the transfer command and filename.
    
2. Locate the related `ftp-data` conversation.
    
3. Select a packet from that data connection.
    
4. Follow its TCP stream.
    
5. Save the displayed data in its original form when possible.
    

### Stream procedure

```text
Select an ftp-data packet
→ Right-click
→ Follow
→ TCP Stream
```

Check whether the stream display needs to be changed from text to raw data before saving a binary file.

---

## 11. Analyze the Live HTTP Traffic

### Purpose

Identify the web server, application, request methods, and downloadable objects.

### Filter syntax

```wireshark
http
```

### Example

```wireshark
http
```

Determine:

- Which host is acting as the web server.
    
- Which application or server software is identified in the headers.
    
- Which HTTP methods appear most frequently.
    
- Whether downloadable files are present.
    
- Whether authentication or other sensitive data is transmitted unencrypted.
    

### Follow a discovered transfer

```text
Select an HTTP packet
→ Right-click
→ Follow
→ TCP Stream
```

If files are present, extract them with:

```text
File → Export Objects → HTTP
```

---

## 12. Additional Helpful Display Filter

The lab’s final review asks how to display only TCP traffic sent by the client. The following derived filter combines the TCP protocol with the suspicious client’s source address.

### Purpose

Show only TCP packets originating from `172.16.10.2`.

### Syntax

```wireshark
tcp && ip.src == <CLIENT_IP>
```

### Example

```wireshark
tcp && ip.src == 172.16.10.2
```

This removes UDP traffic and TCP packets sent in the opposite direction.

---

## 13. Recommended Investigation Workflow

Use the tools in this order:

1. Open the supplied `.pcap` file.
    
2. Review the unfiltered packet list.
    
3. Open **Statistics → Conversations** to identify active endpoints.
    
4. Apply the `http` filter.
    
5. Locate `GET` requests and `200 OK` responses.
    
6. Follow suspicious TCP streams.
    
7. Look for `JFIF`, PNG, content-type, and filename indicators.
    
8. Export HTTP objects.
    
9. Connect to the live lab using `xfreerdp`.
    
10. Capture traffic on `ENS224`.
    
11. Focus on `172.16.10.2` and `172.16.10.20`.
    
12. Apply `ftp` to inspect authentication and transfer commands.
    
13. Apply `ftp-data` to locate transferred file contents.
    
14. Follow and save relevant FTP or HTTP streams.
    
15. Preserve the capture and extracted evidence without modification.
    

## Key Takeaway

Wireshark display filters reduce a large packet capture to the traffic relevant to an investigation. Following TCP streams reconstructs conversations, while HTTP object export and FTP data analysis can recover complete files transferred across unencrypted connections.
