# Internet Protocol Troubleshooting Commands

## Overview

I completed a comprehensive networking troubleshooting lab where I practiced essential command-line utilities used by network administrators to diagnose connectivity issues, identify network paths, and validate service availability. Working through real-world customer scenarios, I gained hands-on experience with tools that map to different layers of the OSI model—from network layer diagnostics through application layer testing. By the end of this lab, I could confidently use five critical troubleshooting commands to identify and resolve networking issues across multiple architectural layers.

## Topics Covered

After completing this lab, I was able to:

- Connect to EC2 instances using SSH and key-pair authentication
- Use `ping` to test Layer 3 (network) connectivity and ICMP reachability
- Use `traceroute` to map network paths and identify latency/packet loss at specific hops
- Use `netstat` to examine Layer 4 (transport) TCP connections and listening services
- Use `telnet` to test Layer 4 (transport) port connectivity and verify firewall rules
- Use `curl` to test Layer 7 (application) HTTP/HTTPS connectivity and validate web service responses
- Map troubleshooting commands to OSI model layers
- Diagnose whether issues originate from local networks, ISPs, or destination servers
- Validate security group and NACL configurations through practical testing

## Introducing the Technologies

### OSI Model and Troubleshooting Commands

Network troubleshooting is most effective when you understand which layer of the Open Systems Interconnection (OSI) model to investigate. Different commands provide visibility into different layers, allowing me to isolate problems systematically.

| OSI Layer | Layer Name | Troubleshooting Commands | What They Test |
|-----------|-----------|--------------------------|-----------------|
| Layer 3 | Network | `ping`, `traceroute` | IP connectivity and routing paths |
| Layer 4 | Transport | `netstat`, `telnet` | TCP/UDP ports and connection states |
| Layer 7 | Application | `curl` | HTTP/HTTPS requests and web service responses |

### Command Categories and Use Cases

These commands serve different diagnostic purposes in networking troubleshooting:

| Command | OSI Layer | Primary Purpose | When to Use |
|---------|-----------|-----------------|------------|
| `ping` | Layer 3 | Test basic IP connectivity | Verify host reachability and latency |
| `traceroute` | Layer 3 | Identify network path and latency per hop | Diagnose where packet loss occurs |
| `netstat` | Layer 4 | View active connections and listening ports | Identify exposed ports or connection issues |
| `telnet` | Layer 4 | Test TCP port connectivity | Verify firewall/security group rules |
| `curl` | Layer 7 | Perform HTTP/HTTPS requests | Test web server responses and headers |

## Tasks

### Task 1: Connect to EC2 Instance via SSH

#### Step 1.1: Access Instance Credentials and Public IP

I accessed the lab credentials panel and downloaded the labsuser.pem file, which was saved to my Downloads folder. I noted the public IP address of the EC2 instance from the credentials window.

#### Step 1.2: Configure SSH Key and Establish Connection

I opened a terminal on my macOS machine and navigated to the Downloads directory:

```bash
cd ~/Downloads
```

I configured the key pair permissions to be read-only (required by SSH for security):

```bash
chmod 400 labsuser.pem
```

I established an SSH connection to the EC2 instance using the key pair, replacing `<public-ip>` with the actual public IP address:

```bash
ssh -i labsuser.pem ec2-user@<public-ip>
```

When prompted, I typed `yes` to accept the host key and establish the connection.

📝 **Note**: The labsuser.pem file was pre-generated during lab setup and stored in my Downloads folder. Key-pair authentication is more secure than password-based SSH authentication.

✅ **Task complete**: Successfully connected to the EC2 instance via SSH.

### Task 2: Practice Layer 3 Troubleshooting with ping and traceroute

#### Step 2.1: Test IP Connectivity with ping

While connected to the EC2 instance, I tested connectivity to a public DNS server using the `ping` command:

```bash
ping 8.8.8.8 -c 5
```

The `-c 5` option limits the command to 5 echo requests (instead of running continuously).

**Sample Output:**
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=10.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=10.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=10.3 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=118 time=10.2 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=118 time=10.1 ms

--- 8.8.8.8 statistics ---
5 packets transmitted, 5 received, 0% packet loss, round-trip min/avg/max = 10.1/10.1/10.3 ms
```

💭 **Consider**: The `ping` command is fundamental for network troubleshooting. If ping fails, it indicates either the host is unreachable or the network security controls (security groups or NACLs) are blocking ICMP traffic. If ping succeeds, basic Layer 3 connectivity is confirmed.

**Use Case Scenario**: A customer reports they cannot reach their database server. Before investigating application logs, I would run `ping <database-ip>` to confirm the server is reachable at the network level. This determines whether the issue is network-related or application-related.

✅ **Task complete**: Successfully pinged 8.8.8.8 with 0% packet loss, confirming network connectivity.

#### Step 2.2: Map Network Path with traceroute

I traced the network path from the EC2 instance to the same public DNS server:

```bash
traceroute 8.8.8.8
```

**Sample Output:**
```
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  ip-172-31-32-1.ec2.internal (172.31.32.1)  0.123 ms  0.087 ms  0.099 ms
 2  100.64.0.1 (100.64.0.1)  1.234 ms  1.201 ms  1.187 ms
 3  * * *
 4  192.0.2.1 (192.0.2.1)  2.456 ms  2.432 ms  2.401 ms
 5  198.51.100.1 (198.51.100.1)  3.567 ms  3.543 ms  3.521 ms
 6  8.8.8.8 (8.8.8.8)  4.678 ms  4.654 ms  4.632 ms
```

💭 **Consider**: Each line in traceroute output represents one "hop"—an intermediate router or server on the path to the destination. The three latency values (in milliseconds) represent three probe packets sent to that hop.

**Reading the Output:**
- Hops 1-2: AWS infrastructure (low latency, under 2 ms)
- Hop 3: The three asterisks (***) indicate a failed hop—this router did not respond to probes
- Hops 4-6: Public internet path to Google's DNS server (increasing latency as distance increases)

**Use Case Scenario**: A customer reports high latency or packet loss when connecting to a remote server. By comparing traceroute output, I can determine:
- If packet loss occurs early in the route (near hop 2-3), the issue is likely AWS or the ISP
- If packet loss occurs later in the route (toward the destination), the issue is likely the destination server or its network provider
- Failed hops (***) usually indicate routers configured not to respond to probes, which is normal and not a failure

✅ **Task complete**: Successfully mapped the network path to 8.8.8.8 using traceroute.

### Task 3: Practice Layer 4 Troubleshooting with netstat and telnet

#### Step 3.1: Examine TCP Connections with netstat

I examined the current TCP connections and listening services on the instance:

```bash
netstat -tp
```

This displays all established TCP connections with the associated process names and PIDs.

**Sample Output:**
```
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 ip-172-31-32-X.ec2.i:22 203.0.113.100:54321   ESTABLISHED 1234/sshd
tcp        0      0 ip-172-31-32-X.ec2.i:22 203.0.113.101:54322   ESTABLISHED 5678/sshd
```

I also examined listening services (ports that accept incoming connections):

```bash
netstat -tlp
```

**Sample Output:**
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      999/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      888/postfix
```

| netstat Option | Purpose | Output Shows |
|----------------|---------|--------------|
| `-t` | TCP connections only | Filters to TCP (excludes UDP) |
| `-p` | Program/process name | Which process owns the connection |
| `-l` | Listening sockets only | Ports that accept incoming connections |
| `-n` | Numeric IP addresses | IP addresses instead of resolved hostnames |

💭 **Consider**: `netstat -tp` shows established connections—if you see unexpected remote addresses or ports, this indicates a potential security issue or compromised system. `netstat -tlp` shows which services are listening—if you see ports that shouldn't be open, this could indicate misconfiguration or unauthorized services.

**Use Case Scenario**: A security scan reveals an unexpected open port on a server. I would run `netstat -tlp` to identify which process is listening on that port and verify whether it should be running.

✅ **Task complete**: Examined TCP connections and listening services using netstat.

#### Step 3.2: Install telnet Utility

The telnet utility was not pre-installed, so I installed it:

```bash
sudo yum install telnet -y
```

The `-y` flag automatically answers "yes" to any prompts.

#### Step 3.3: Test Port Connectivity with telnet

I tested TCP connectivity to port 80 (HTTP) on Google's web server:

```bash
telnet www.google.com 80
```

**Sample Output (Successful Connection):**
```
Trying 142.251.32.14...
Connected to www.google.com.
Escape character is '^]'.
HTTP/1.0 400 Bad Request
Content-Length: 1478
Content-Type: text/html; charset=UTF-8
...
```

The key indicators of success:
- `Connected to www.google.com` message appears
- Terminal shows data being transmitted from the server
- Connection remains open until I manually close it

To exit telnet, I pressed `Ctrl+]` to enter escape mode, then typed `quit`:

```
^]
telnet> quit
Connection closed.
```

| telnet Connection Outcome | Meaning |
|---------------------------|---------|
| `Connected to [host]` | TCP port is open and accepting connections; no firewall blocking |
| `Connection refused` | TCP port is closed or security group/NACL is denying connections |
| `Connection timed out` | No network route exists or host is unreachable (network-level issue) |

💭 **Consider**: Telnet tests the **Layer 4 (Transport) connectivity only**—it confirms the TCP connection succeeds. It does NOT verify the service is working correctly (that's Layer 7 testing with curl). If telnet succeeds but the application doesn't respond as expected, the issue is application-level, not network-level.

**Use Case Scenario**: A customer claims their security group blocks port 80, but they want to verify it's actually closed. I would run `telnet <instance-ip> 80`. If the connection is refused, the port is closed (security group rule is working). If the connection succeeds, the port is open (security group rule may not be configured as expected).

✅ **Task complete**: Successfully tested port connectivity using telnet.

### Task 4: Practice Layer 7 Troubleshooting with curl

#### Step 4.1: Test Web Service Connectivity with curl

I tested HTTP/HTTPS connectivity to the AWS website and examined the response headers:

```bash
curl -vLo /dev/null https://aws.com
```

| curl Option | Purpose |
|------------|---------|
| `-v` | Verbose mode—shows headers and communication details |
| `-L` | Follow redirects—if the server redirects (like HTTP → HTTPS), follow automatically |
| `-o /dev/null` | Discard response body—only show headers, not full HTML content |

**Sample Output:**
```
*   Trying 54.239.28.30:443...
* Connected to aws.com (54.239.28.30) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations
* TLSv1.3 (OUT), TLS handshake, Client Hello (1):
...
> GET / HTTP/1.1
> Host: aws.com
> User-Agent: curl/7.76.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: text/html; charset=UTF-8
< Content-Length: 45678
< Connection: keep-alive
...
```

Key indicators in the output:
- `HTTP/1.1 200 OK` indicates the server successfully processed the request
- Response headers show content type, length, and other metadata
- TLS handshake details show HTTPS encryption is working

**Other Useful curl Options:**

| Option | Purpose |
|--------|---------|
| `-I` | Head request only—fetches headers without response body |
| `-i` | Include headers in output—shows both headers and body |
| `-k` | Ignore SSL certificate errors—useful for self-signed certificates in testing |
| `-X POST` | Specify HTTP method (GET, POST, PUT, DELETE, etc.) |

💭 **Consider**: The `curl` command operates at Layer 7 (Application), so it tests the complete end-to-end path including the web server's ability to process HTTP requests. If curl succeeds, all lower layers (transport, network, physical) must also be functioning correctly.

**Successful Response Codes:**
- `200 OK` - Request succeeded
- `301/302` - Redirect (handled automatically with `-L` flag)
- `404 Not Found` - Resource not found (but server is responding)

**Failure Indicators:**
- `Connection refused` - TCP connection failed (network/firewall issue)
- `Connection timed out` - No response (routing or host unreachable)
- `SSL certificate error` - HTTPS certificate problem (use `-k` to ignore in testing)
- `500 Server Error` - Web server is responding but application has issues

**Use Case Scenario**: A customer's Apache web server is running, but they're unsure if it's responding to HTTP requests. I would run `curl http://<server-ip>` to verify the server responds with a 200 OK status code. If it returns a 500 error, the application layer has issues. If the connection times out, the server isn't reachable at all.

✅ **Task complete**: Successfully tested web service connectivity using curl and received HTTP 200 OK response.

## Conclusion

I successfully practiced and demonstrated proficiency with five critical network troubleshooting commands across multiple OSI layers:

- ✅ Connected to an EC2 instance via SSH using key-pair authentication
- ✅ Used `ping` to test Layer 3 IP connectivity and confirmed network reachability
- ✅ Used `traceroute` to map the network path and identify latency at each hop
- ✅ Used `netstat` to examine established TCP connections and listening services
- ✅ Used `telnet` to test Layer 4 port connectivity and verify firewall rules
- ✅ Used `curl` to test Layer 7 HTTP/HTTPS connectivity and validate web server responses
- ✅ Mapped each command to its corresponding OSI layer and use case scenario
- ✅ Demonstrated ability to diagnose whether issues originate from local networks, ISPs, or destination servers

By understanding which command to use at each OSI layer, I can now efficiently troubleshoot customer networking issues, isolate problems to their source, and validate fixes systematically.

## Additional Resources

- [ping Command Documentation](https://linux.die.net/man/8/ping)
- [traceroute Command Documentation](https://linux.die.net/man/8/traceroute)
- [netstat Command Documentation](https://linux.die.net/man/8/netstat)
- [telnet Command Documentation](https://linux.die.net/man/1/telnet)
- [curl Command Documentation](https://curl.se/docs/manpage.html)
- [OSI Model Overview](https://en.wikipedia.org/wiki/OSI_model)
- [TCP/IP Basics](https://docs.aws.amazon.com/whitepapers/latest/aws-caf-security-perspective/network-security.html)
- [AWS Network Troubleshooting Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html)
