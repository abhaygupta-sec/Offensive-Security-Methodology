# Pivoting, Tunneling & Port Forwarding

Pivoting allows a compromised host to be used as an intermediary for reaching systems, services, or networks that are not directly accessible from the attacker's machine.

The objective is to understand **network reachability and traffic direction**, then select the appropriate tunneling or forwarding technique.

---

## Methodology

My general pivoting workflow follows:

```text
Initial Access
      ↓
Enumerate Network Interfaces
      ↓
Identify Reachable Networks
      ↓
Identify Internal Targets
      ↓
Determine Traffic Direction
      ↓
Select Pivot Technique
      ↓
Configure Tunnel / Forward
      ↓
Verify Connectivity
      ↓
Enumerate Internal Services
      ↓
Continue Attack Path

The core principle is:

A compromised host can become a new attack platform.

1. Identify the Pivot

After obtaining access to a host, determine whether it has access to networks or services that are unavailable from the attacking machine.

Linux
ip addr
ip route

Additional useful information:

hostname
cat /etc/hosts
Windows
ipconfig /all
route print

Review:

Network interfaces
IP addresses
Subnets
Default gateways
Routing table
Internal DNS
Hostnames
Other reachable networks
2. Identify the Internal Network

The first question is:

What can the compromised host reach that I cannot reach directly?

Example:

Attacker
   |
   | Direct Access
   v
Compromised Host
   |
   | Internal Network
   v
10.10.20.0/24
   |
   +---- 10.10.20.10
   +---- 10.10.20.20
   +---- 10.10.20.30

The compromised host may have multiple network interfaces:

Internet / Attacker Network
          |
          v
     Pivot Host
          |
          v
   Internal Network

This is the fundamental pivoting scenario.

3. Understand Traffic Direction

Before configuring a tunnel, determine:

Who can reach whom?

For example:

Attacker → Pivot
Pivot → Internal Target

If the attacker cannot directly reach the internal target, traffic must be routed through the pivot.

This can be represented as:

Attacker
   ↓
Tunnel
   ↓
Pivot
   ↓
Internal Target

Traffic direction determines which forwarding technique should be used.

4. SSH Local Port Forwarding

SSH local forwarding is useful when one specific internal service needs to be accessed.

General syntax:

ssh -L <LOCAL_PORT>:<INTERNAL_TARGET>:<REMOTE_PORT> <USER>@<PIVOT>

Example:

ssh -L 8080:10.10.20.10:80 user@<PIVOT>

The resulting path is:

127.0.0.1:8080
      ↓
SSH Tunnel
      ↓
Pivot
      ↓
10.10.20.10:80

The application can then be accessed locally through:

http://127.0.0.1:8080
When to Use

Use local forwarding when:

One specific internal service is required
The internal target is reachable from the pivot
A full network proxy is unnecessary
5. SSH Dynamic Port Forwarding

Dynamic forwarding creates a SOCKS proxy.

Example:

ssh -D 9050 <USER>@<PIVOT>

Traffic can then be routed through the SOCKS proxy.

Conceptually:

Attacker
   ↓
SOCKS Proxy
   ↓
SSH Tunnel
   ↓
Pivot
   ↓
Internal Network
When to Use

Use dynamic forwarding when:

Multiple internal services need to be accessed
The destination changes frequently
Proxy-aware tools can be used
A full network route is unnecessary
6. SSH Remote Port Forwarding

Remote forwarding is useful when the connection direction requires the pivot or internal host to connect back toward the attacker's machine.

General syntax:

ssh -R <REMOTE_PORT>:<TARGET>:<TARGET_PORT> <USER>@<HOST>

Conceptually:

Internal / Pivot Host
        ↓
SSH Remote Forward
        ↓
Attacker
        ↓
Listener / Service

The important consideration is:

Which side can establish the connection?

Remote forwarding is useful when the normal attacker-to-target direction is blocked.

7. SSH Pivoting Decision Tree
Need one internal service?
        |
       YES
        ↓
SSH Local Port Forward
Need multiple internal services?
        |
       YES
        ↓
SSH Dynamic SOCKS Proxy
Internal host needs to reach attacker?
        |
       YES
        ↓
SSH Remote Port Forward
8. Meterpreter Pivoting

Meterpreter can be used to route traffic through a compromised session.

General workflow:

Compromised Session
        ↓
Identify Internal Network
        ↓
Add Route
        ↓
Verify Route
        ↓
Scan / Access Internal Target

Example:

meterpreter > ipconfig

Add a route:

meterpreter > run autoroute -s <INTERNAL_SUBNET>

Review routes:

meterpreter > run autoroute -p

The exact route should correspond to the internal network discovered on the compromised host.

9. SOCKS Proxy Pivoting

A SOCKS proxy allows compatible tools to send traffic through the compromised host.

Conceptually:

Tool
  ↓
SOCKS Proxy
  ↓
Compromised Host
  ↓
Internal Network
  ↓
Target

This is useful when several internal services need to be accessed without creating a separate local port forward for every service.

10. Chisel

Chisel can be used to create TCP tunnels between systems.

The general model is:

Attacker
   ↕
Chisel Tunnel
   ↕
Pivot
   ↕
Internal Network

Chisel can operate in different client/server configurations depending on which host can initiate the connection.

The correct setup depends on:

Which host can reach the other
Firewall restrictions
Listener availability
Whether a reverse connection is required
Which internal service needs to be accessed
11. Reverse Tunneling

Reverse tunneling is useful when the compromised host can make outbound connections to the attacker but the attacker cannot directly connect to the compromised host.

Conceptually:

Attacker
    ↑
    | Existing / Allowed Connection
    |
Compromised Host
    |
    ↓
Internal Target

The tunnel reverses the normal connection direction.

This is particularly useful when:

Inbound connections are blocked
Outbound connections are allowed
NAT prevents direct access
The pivot can reach the internal target
12. Windows Port Forwarding

Windows hosts can also be used to forward traffic.

One available approach is Windows netsh port forwarding.

Conceptually:

Attacker
   ↓
Windows Pivot
   ↓
Internal Target

Before configuring forwarding, identify:

ipconfig
route print

Then determine:

Listening address
Local port
Internal destination
Destination port
Required routing
13. Socat

socat can be used to create TCP relays and port forwards.

General concept:

Listener
   ↓
socat
   ↓
Destination

Example structure:

socat TCP-LISTEN:<LOCAL_PORT>,fork TCP:<TARGET>:<TARGET_PORT>

The exact configuration depends on which host runs the listener and which side can reach the destination.

14. Port Forwarding vs SOCKS

The choice depends on the objective.

Technique	Best Use
SSH -L	One known internal service
SSH -D	Multiple destinations through SOCKS
SSH -R	Reverse connection requirements
Meterpreter routing	Internal network access through a Meterpreter session
Chisel	Flexible TCP tunneling
netsh	Windows-based forwarding
Socat	Simple TCP relays / forwarding
15. Verify the Tunnel

A tunnel is not complete until connectivity is verified.

For example:

curl http://127.0.0.1:<LOCAL_PORT>

Or:

nc -vz 127.0.0.1 <LOCAL_PORT>

For SOCKS-based access, verify that the tool actually uses the proxy.

The troubleshooting workflow is:

Tunnel Created
      ↓
Listener Confirmed
      ↓
Route Confirmed
      ↓
Connection Attempt
      ↓
Target Service Responds?
      ↓
Continue Enumeration
16. Internal Enumeration After Pivoting

Once the tunnel works, treat the internal network as a new attack surface.

Look for:

Web services
SMB
LDAP
Kerberos
SSH
RDP
Databases
Management interfaces
Internal APIs
Domain Controllers

The workflow becomes:

Pivot
  ↓
Internal Network Discovery
  ↓
Service Enumeration
  ↓
Credential Discovery
  ↓
Initial Access
  ↓
Privilege Escalation
  ↓
Additional Pivot

A pivot is therefore not the end of an attack path.

It often creates the next one.

17. Multi-Level Pivoting

Sometimes the first compromised host cannot directly reach the final target network.

Example:

Attacker
   |
   v
Pivot 1
   |
   v
Pivot 2
   |
   v
Internal Network
   |
   v
Target

Each pivot provides access to another network segment.

The methodology remains:

Compromise
   ↓
Enumerate Routes
   ↓
Identify Next Network
   ↓
Establish Tunnel
   ↓
Verify Connectivity
   ↓
Continue Enumeration
18. Troubleshooting

When a tunnel does not work, check each layer separately.

1. Can the pivot reach the target?
Pivot → Target
2. Can the attacker reach the pivot?
Attacker → Pivot
3. Is the listener running?
Listener → Confirm Port
4. Is the forwarding rule correct?
Local Port
      ↓
Tunnel
      ↓
Target IP
      ↓
Target Port
5. Is the application using the tunnel?

A SOCKS proxy does nothing if the tool is not configured to use it.

19. Common Mistakes

Avoid:

Creating a tunnel without understanding the network
Using the wrong traffic direction
Forwarding to the wrong IP address
Forgetting the target port
Assuming the pivot can reach every internal host
Forgetting to verify routes
Forgetting to verify listeners
Creating a SOCKS proxy but not configuring tools to use it
Attempting exploitation before confirming connectivity
Forgetting that internal services may require different authentication
Ignoring additional networks exposed by a second pivot
20. Pivoting Decision Tree
Compromised Host
       |
       v
Enumerate Interfaces / Routes
       |
       v
Identify Internal Network
       |
       v
Can Attacker Reach Target Directly?
       |
    +--+--+
    |     |
   YES    NO
    |     |
    v     v
Direct   Pivot Required
Access       |
             v
      Determine Objective
             |
       +-----+------+
       |            |
   One Service   Multiple Services
       |            |
       v            v
    SSH -L       SOCKS / Tunnel
       |            |
       +-----+------+
             |
             v
       Verify Connectivity
             |
             v
       Internal Enumeration
21. Pivoting Mindset

The objective is not:

Get Shell
   ↓
Run Chisel
   ↓
Done

The objective is:

Compromise Host
      ↓
Understand Network Position
      ↓
Identify Reachable Networks
      ↓
Determine Traffic Direction
      ↓
Choose Forwarding Method
      ↓
Establish Tunnel
      ↓
Verify Connectivity
      ↓
Enumerate Internal Services
      ↓
Continue Attack Path

The most important question is:

What can this compromised host reach that I cannot reach directly?

22. Reporting Pivoting

Every pivot should be documented clearly.

Record:

SOURCE
Which host initiated the connection?

PIVOT
Which compromised host was used?

DESTINATION
Which internal host/service was accessed?

METHOD
Which tunneling / forwarding technique was used?

REASON
Why was this technique selected?

RESULT
What internal access became available?

NEXT STEP
What was discovered after pivoting?

Example:

External Attacker
      ↓
Compromised Web Server
      ↓
SSH / SOCKS Tunnel
      ↓
Internal Network
      ↓
Internal Web Service
      ↓
Further Enumeration
Related Projects
HTB Writeups

Practical machine assessments demonstrating pivoting, tunneling, and multi-stage attack paths.

HTB-Writeups

Offensive Security Methodology

See the repository root for the broader penetration-testing methodology.

Responsible Use

All techniques documented here are intended for:

Authorized penetration tests
Security laboratories
Hack The Box and other CTF environments
Security research
Systems owned or explicitly authorized for testing

Never use pivoting or tunneling techniques against systems without authorization.
