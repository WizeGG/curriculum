# Python/Linux Project — Red vs. Blue Capture-the-Flag (CTF)

- **Duration:** `TBD`
- **Challenge Type:** `Blue Teams: Two Groups of 5 | Red Teams: Two Groups of 4`
- **Deliverable:** `GitHub repository + Web app + Written reports`

---

## Context

You have completed the Networking modules and Python curriculum. Now it is time to apply your knowledge in a competitive, real-world scenario.

In this project, your team will be divided into competing roles:

- **Blue Team (Defenders/Web Developers):** Build a secure web application (Flask or Node.js) and intentionally leave one specific vulnerability or open port for the Red Team to discover. Document your defensive hardening process.
- **Red Team (Attackers/Pentesters):** Build a Python port scanner to map the network, identify exposed services, discover vulnerabilities, and document your offensive findings.

Before any code is written, both teams will **observe** what port scanning looks like at the network level using Wireshark. This is intentional. Understanding the network-level impact of your tools is as important as building them.

---

## Lab Setup

All activities must occur **exclusively within an isolated lab environment** — never on the internet or on networks you do not own.

### What You Need

**Minimum requirement: 2 Virtual Machines**

- **Blue Team Server** — a machine to host the web application (Ubuntu Server or similar with Flask/Node.js runtime). The Red Team will attack this.
- **Red Team Scanner** — a machine with Python 3 for running the port scanner and reconnaissance tools. This machine also runs Wireshark to analyze network traffic (keeping resource usage efficient).

Both machines connect to the same isolated network so they can communicate with each other, but with no access to the internet.

**Alternative setup (if lab resources are limited):**
Teams can operate on the **same host machine** using containers (Docker) or separate processes, provided network isolation is properly maintained via an **Internal Network** mode. However, the two-VM setup is recommended for the most realistic lab experience.

### Choosing a Hypervisor

| Your machine | Recommended hypervisor (Free Download Links) |
|---|---|
| **Windows / Linux (x86/x64)** | [**VirtualBox**](https://www.virtualbox.org/wiki/Downloads) |
| **macOS (Intel)** | [**VirtualBox**](https://www.virtualbox.org/wiki/Downloads) |

### Network Configuration

All VMs are recommended but not required to use **Internal Network** mode with the network name of your choice. Follow these steps for your hypervisor:

**Step 1: Configure the Network in Your Hypervisor**

**If using VirtualBox:**
1. Open VirtualBox → **File → Tools → Network Manager**
2. Click **Create** to add a new internal network
3. Name it: `network name of your choice`
4. Click **Apply**

**Step 2: Apply Internal Network to Both VMs**

1. For each VM (Blue Team Server and Red Team Scanner):
   - Open VM **Settings → Network**
   - Set **Adapter 1** to: `Internal Network`
   - Select network name: `network name of your choice`
   - Click **OK**

2. Repeat for the second VM — **both must use the same network name**

**⚠️ CRITICAL:** Do not use Bridged or NAT mode. These modes will cause connectivity issues and may trigger security alerts on campus networks. Internal Network is the only approved configuration for this lab.

### Verify Connectivity

1. Boot all team machines
2. On the Blue Team Server, find its IP: `ip addr show` or `ifconfig`
3. From the Red Team Scanner, test connectivity: `ping <blue-server-ip>`

If you get a response, your lab is ready to proceed.

---

## Phase 1 — Observe Before You Build

Before any code is written, both teams must use **[Wireshark](https://www.wireshark.org)** to capture and analyze network traffic. This demonstrates what reconnaissance and security scanning looks like from the wire.

**[Quick tutorial on How to use Wireshark](https://www.varonis.com/blog/how-to-use-wireshark)**
**[More advanced tutorial](https://hackertarget.com/wireshark-tutorial-and-cheat-sheet/)**

### Steps for Both Teams:

1. On the Red Team Scanner, open Wireshark and begin capturing on the primary network interface
2. Run a basic network discovery: `ping <blue-server-ip>`
3. Simulate a port scan attempt (Red Team will execute this later): Observe the traffic pattern
4. For the Blue Team: Observe what normal web traffic looks like vs. scanning traffic
5. Stop the capture and answer the following questions — answers go into your **Design Report**:
   - What protocol is used for network discovery (ARP, ICMP)? What does the traffic pattern look like?
   - What does a connection attempt to an **open** port look like? (TCP handshake)
   - What does a connection attempt to a **closed** port look like? How is it different?
   - Can you spot the scanning pattern? What makes it recognizable as reconnaissance?
   - What defensive indicators would alert a Blue Team to an ongoing scan?

Include **at least one annotated Wireshark screenshot** in both teams' reports showing the difference between scanning traffic and legitimate traffic.

---

## Phase 2 — Team Roles & Implementation

### Role: Red Team (Penetration Testers)

**Objective:** Discover the Blue Team's network, identify exposed services, and locate vulnerabilities.

#### MUST-HAVE:

1. **Build a Python Port Scanner** that:
   - Accepts a target IP and a port range as input
   - Performs network discovery to map active hosts
   - Reports each port as `OPEN`, `CLOSED`, or `FILTERED`
   - Supports multiple scan types (TCP Connect, SYN, UDP — at least two types)
   - Saves results to a structured file (`.json`, `.csv`, or `.txt`)
   - Includes proper error handling for timeouts and unreachable hosts

2. **Reconnaissance Report** containing:
   - Network map showing discovered hosts and open ports
   - Identified services running on each port
   - Hypothesis of the vulnerability or weak point left by Blue Team
   - Screenshot evidence from your scanner output

3. **Exploitation Report** documenting:
   - Attempt to access or interact with the identified vulnerable service
   - Documentation of what the vulnerability exposes (e.g., directory traversal, SQL injection, exposed endpoint)
   - Proof-of-concept code or steps to reproduce the vulnerability
   - Risk assessment (CVSS or simplified severity rating)

4. **Defensive Analysis Report** covering:
   - What scanning patterns appear in network logs?
   - How would a Blue Team detect your reconnaissance activity?
   - Recommended mitigations based on your attack methods

---

### Role: Blue Team (Web Developers)

**Objective:** Build and intentionally secure (with one deliberate vulnerability) a web application. Document your hardening process.

#### MUST-HAVE:

1. **Build a Web Application** using Flask or Node.js that:
   - Runs on a non-standard port (e.g., 8080, 9000, or higher) to avoid obvious discovery
   - Implements basic security controls (HTTPS, input validation, authentication for sensitive endpoints)
   - **Intentionally includes ONE specific vulnerability** for Red Team to discover (examples below):
     - A public endpoint that leaks system information (`/debug`, `/api/version`, `/admin`)
     - A weak authentication endpoint (`/login` with default credentials)
     - Path traversal vulnerability (`/download?file=../../etc/passwd`)
     - Exposed API endpoint listing database records
     - Hardcoded credentials in comments or environment variables
   - Logs all access attempts for your defensive analysis

2. **Application Documentation** including:
   - Code architecture and design decisions
   - List of implemented security controls
   - **Clear documentation of the intentional vulnerability**
   - How each security control was implemented and why

3. **Hardening Report** documenting:
   - Security controls implemented (input validation, output encoding, authentication, HTTPS, CORS policies, rate limiting)
   - Why each control was chosen
   - Defense-in-depth strategies employed
   - Monitoring and logging mechanisms in place

4. **Incident Response Report** containing:
   - Evidence of Red Team's reconnaissance (Wireshark captures showing scan traffic)
   - Log analysis showing exploitation attempts
   - How you detected the attack (IDS signatures, unusual traffic patterns, application logs)
   - Recommended fixes for the intentional vulnerability if this were production

---

## Key Implementation Details

### For Red Team: Scanner Design

**Skills Being Applied:**
- Loops & Collections — iterating over IP ranges and port lists
- Functions & Typing — clean, reusable, type-hinted code
- Exception Handling — timeouts, unreachable hosts, invalid input
- File Handling — saving structured scan results
- Networking knowledge — understanding TCP/UDP handshakes, port states
- Scan Types — SYN scans, Connect scans, UDP scans, and their differences

**Example Scanner Features:**
```
python scanner.py --target 192.168.1.0/24 --ports 1-10000 --type syn --output results.json
python scanner.py --target 192.168.1.50 --ports 80,443,8080,9000 --type connect
```

### For Blue Team: Web App Design

**Skills Being Applied:**
- Web framework architecture (Flask/Node.js)
- Security best practices (OWASP Top 10)
- Logging and monitoring
- Configuration management (environment variables, secrets)
- Testing and validation

**Example Intentional Vulnerabilities (Choose One):**

| Vulnerability | Example | Why It's Good for CTF |
|---|---|---|
| **Exposed Debug Endpoint** | `GET /api/debug` returns system info (Python version, dependencies, environment) | Easy to discover via port scan, realistic in development |
| **Default Credentials** | Admin panel at `/admin` with credentials in README or hardcoded | Tests if Red Team explores obvious locations |
| **Information Disclosure** | `GET /api/version` leaks exact version numbers | Allows version-based vulnerability research |
| **Path Traversal** | `GET /download?file=../../etc/passwd` reads arbitrary files | Tests understanding of input validation |
| **Exposed API** | `GET /api/users` returns user list without authentication | Tests if Red Team explores API endpoints |

---

## Deliverables

### Red Team:

- ✅ **GitHub Repository** containing:
  - `scanner.py` (or modular `scanner/` directory) — fully functional port scanner
  - `requirements.txt` — all dependencies with versions
  - `README.md` — usage instructions and features
  - `/reports` folder containing:
    - `01-design-report.md` — architecture, libraries, Wireshark findings
    - `02-reconnaissance-report.md` — network map, discovered services, vulnerability hypothesis
    - `03-exploitation-report.md` — vulnerability details, PoC code, severity assessment
    - `04-defensive-analysis.md` — detection signatures, recommended mitigations
  - Supporting files: Wireshark captures (`.pcap`), scan output samples
- ✅ **Working Scanner** — executable code that successfully identifies open ports
- ✅ **Live Presentation** — demonstrate the scanner discovering the Blue Team's vulnerable service

### Blue Team:

- ✅ **GitHub Repository** containing:
  - `app.py` or `index.js` — the web application code
  - `requirements.txt` or `package.json` — all dependencies
  - `README.md` — setup instructions, security controls, **vulnerability disclosure** (for grading)
  - `Dockerfile` (optional) — reproducible deployment
  - `/reports` folder containing:
    - `01-design-report.md` — architecture, security controls, intentional vulnerability rationale
    - `02-hardening-report.md` — security implementations, defense-in-depth strategy
    - `03-incident-response.md` — evidence of attack, detection methods, log analysis
  - `/logs` folder — application logs and network captures showing the attack
- ✅ **Running Web Application** — accessible on the isolated lab network
- ✅ **Live Presentation** — explain your security controls and how you detected the Red Team's attack

### Both Teams:

- ✅ **Wireshark Captures** — at least one annotated `.pcap` file showing reconnaissance vs. legitimate traffic
- ✅ **Team Reflection** — how the exercise illustrates real-world offense vs. defense dynamics

---

## Evaluation Criteria

### Red Team:

| Criteria | Excellent | Good | Needs Work |
|---|---|---|---|
| **Scanner Functionality** | Correctly identifies open/closed/filtered ports with multiple scan types | Identifies ports but limited scan types or occasional false positives | Produces incorrect results or fails on edge cases |
| **Code Quality** | Well-structured, type-hinted, thoroughly tested | Functional but lacks organization or error handling | Difficult to follow, no error handling |
| **Reconnaissance** | Clearly maps the network, identifies the vulnerability, provides strong hypothesis | Maps network, identifies service, weak hypothesis | Misses key details, vague findings |
| **Exploitation** | Successful PoC demonstrating the vulnerability with clear evidence | Attempts exploitation, partial success | Cannot exploit or unclear methodology |
| **Documentation** | Clear, detailed, Wireshark evidence included, defensive analysis thorough | Complete but lacks detail or evidence | Missing sections or superficial analysis |

### Blue Team:

| Criteria | Excellent | Good | Needs Work |
|---|---|---|---|
| **Application Quality** | Secure, runs reliably, intentional vulnerability is discoverable | Mostly functional, security controls present, vulnerability present | Crashes, security issues beyond the intentional one, vulnerability unclear |
| **Vulnerability Design** | Clever, realistic, requires Red Team reconnaissance | Straightforward to discover, realistic | Too obvious or not realistic |
| **Security Controls** | Multiple overlapping controls, defense-in-depth strategy evident | Basic controls implemented, reasoning clear | Minimal controls, weak implementation |
| **Detection & Logging** | Successfully detects Red Team's attack, logs are detailed and analyzed | Detects attack, logs present but minimal analysis | Misses detection or no logs |
| **Documentation** | Clear explanation of controls, incident response thorough with evidence | Complete documentation, adequate incident response | Incomplete or unclear documentation |

---

## Important Notes & Warnings

> ⚠️ **All testing and scanning must occur EXCLUSIVELY within your isolated lab VMs. Scanning any IP address you do not own without authorization is illegal and unethical.**

> ⚠️ **Ethics & Legality:**
> - This exercise simulates real attack/defense scenarios. The skills you develop must never be used against systems you don't own or without explicit written authorization.
> - Unauthorized port scanning, exploitation attempts, and network reconnaissance are crimes in most jurisdictions (Computer Fraud and Abuse Act in the US, similar laws globally).
> - This lab is for **educational purposes only** in a controlled environment.

> ⚠️ **Blue Team Vulnerability Disclosure:**
> - Document your intentional vulnerability clearly in your README so graders understand it was deliberate.
> - In real-world scenarios, vulnerabilities should be reported to developers, not exploited. Always follow responsible disclosure principles.

> ⚠️ **Red Team Attack Rules:**
> - Only target the isolated Blue Team server within the lab.
> - Do not attempt to compromise other machines or networks.
> - Document your methodology—this is a learning exercise, not a crime.

---

## Success Criteria

**All teams win if:**
- ✅ They produce high-quality, well-documented code
- ✅ They demonstrate learning and understanding of offense/defense principles
- ✅ They can explain their methodology clearly

### The Real Goal: Teamwork & Learning

This exercise is fundamentally about **learning both offensive and defensive perspectives** through hands-on practice. While the teams are competing, **you are encouraged to coordinate** to ensure the Red Team can discover the Blue Team's intentional vulnerability.

The success of this lab is NOT about one team "winning" over the other — it's about:
- **Red Team:** Learning how to conduct reconnaissance, build tools, and think like a penetration tester
- **Blue Team:** Learning how to secure applications, detect attacks, and think like a defender
- **Both Teams:** Understanding that offense and defense are complementary skills, and that **collaboration in a lab environment is how we learn together**

Think of it this way: If the Red Team struggles to find the vulnerability, they're learning less about scanning techniques. If the Blue Team can't detect the attack, they're missing a valuable defensive lesson. Coordinate to make sure both teams extract maximum educational value from the exercise.

**Teamwork makes this exercise valuable. Competition makes it fun. Both together = success.**

---

## Resources

- **Network Scanning:** [nmap.org](https://nmap.org), [HackerTarget Nmap Tutorial](https://hackertarget.com/nmap-tutorial/)
- **Python Sockets:** [Python docs: socket module](https://docs.python.org/3/library/socket.html)
- **Flask Security:** [Flask Security Best Practices](https://flask.palletsprojects.com/)
- **OWASP Top 10:** [OWASP Top 10 Web Application Risks](https://owasp.org/www-project-top-ten/)
- **Wireshark:** [Wireshark User Guide](https://www.wireshark.org/docs/)
