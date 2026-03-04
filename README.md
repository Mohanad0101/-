# Basic Network Socket Programming — Lab

This lab teaches you how programs exchange data over a network using **sockets** in Python: TCP echo server and client, pickle, UDP echo, and a simple JSON bank. The instructions below are the **full lab content**. You will **clone** (or create) a repo, **implement** the code in each folder, **record results**, and **push to your remote** so you can submit **source code + results** to your instructor.

**Source:** [fa-python-network/1_echo_server](https://github.com/fa-python-network/1_echo_server); lecture on socket programming (koroteev.site).

---

## Workflow & submission — read this first

### What you must deliver

1. **Source code** — Your implemented `server.py` and `client.py` (or only `server.py` where stated) in folders `01_tcp_echo_one` through `10_assignment6_auto_port`.
2. **Results** — Evidence that your programs run. Put run outputs (or screenshots) in the **`results/`** folder (see below).
3. **Identification** — Fill in **`STUDENT_INFO.txt`** with your **full name** and **group** (or student ID).

You will **push the whole lab** (code + results + README + STUDENT_INFO) to **your own remote repository** and submit the **repo URL** to your instructor.

---

### Option A: Clone the instructor’s repo, then push to your remote

Replace `INSTRUCTOR_REPO_URL` with the actual URL your instructor gives you (e.g. `https://github.com/instructor/basic_socket_lab`).

```bash
# 1. Clone the lab repository
git clone INSTRUCTOR_REPO_URL
cd basic_socket_lab

# 2. Implement code in each folder (01_ … 10_). Run server then client; capture results.
# 3. Create results folder and add run outputs (see "Results" below).
# 4. Fill in STUDENT_INFO.txt (name, group).

# 5. Create your own empty repo on GitHub/GitLab (e.g. my-socket-lab), then:
git add .
git commit -m "Complete socket lab: source code and results"
git remote rename origin upstream
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

If the default branch is `master` instead of `main`:

```bash
git push -u origin master
```

Submit your repo URL (e.g. `https://github.com/YOUR_USERNAME/YOUR_REPO`) to your instructor.

---

### Option B: Create a new local repo and push to your remote

If you received the lab as a ZIP or folder (no git history):

```bash
# 1. Go into the lab folder (e.g. basic_socket_lab)
cd basic_socket_lab

# 2. Initialize a new repository
git init

# 3. Implement code in each folder; add results to results/; fill STUDENT_INFO.txt.

# 4. Add all files, commit, and push to your remote
git add .
git commit -m "Complete socket lab: source code and results"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

Create the empty repo `YOUR_REPO` on GitHub/GitLab first. Submit the repo URL to your instructor.

---

### Results folder (required)

Create a folder named **`results/`** in this repo. For each step you implement, add evidence that it runs:

- **Option 1 (recommended):** Text files with terminal output, e.g. `results/01_tcp_echo_one.txt` containing the server and client output when you run Step 1.
- **Option 2:** Screenshots (e.g. `results/01_tcp_echo_one.png`) showing the server and client running.

Include at least outputs (or screenshots) for: **01**, **02**, **06** (UDP), and **07** (bank). You may include more. This shows the instructor your source code runs correctly.

---

## What you will learn

- What a **socket** is and how **server** and **client** roles differ (bind/listen/accept vs connect).
- How a **TCP** connection is established and how data is sent and received as **bytes**.
- Why **encode()** and **decode()** are needed for text and how to handle the **byte stream** (no built-in messages).
- How to make a server **reusable** and how to use **timeouts** and **SO_REUSEADDR**.
- How to send a **Python object** (e.g. a dictionary) using **pickle** and **JSON** over TCP.
- How **UDP** differs from TCP (no connection; **recvfrom** / **sendto**).

---

## How to use this lab

1. **Read** this README from top to bottom — especially **Core concepts** and the **Folder map**.
2. **Implement** the server and client in each folder in order (`01_` → … → `10_`). Run the **server** first, then the **client** in another terminal.
3. **Ports:** Use **9090** for TCP echo and pickle, **9091** for UDP echo, **9092** for the bank.
4. **Fill in `STUDENT_INFO.txt`** (name and group).
5. **Add results** to the `results/` folder (outputs or screenshots).
6. **Push** the whole lab to your remote repo and submit the repo URL to your instructor.

---

## Contents

| Section | Content |
|---------|---------|
| **1** | Core concepts (read this to understand socket programming) |
| **2** | Practical steps (TCP echo 1–4, pickle 5, UDP echo 6, simple bank 7) |
| **2b** | Independent assignments 4, 5, 6 (port/host, log file, auto port) |
| **2c** | Complete the code (exercises with hints) |
| **3** | Control questions and answers |
| **4** | Full list of assignments for independent work |
| **5** | Quick reference |

---

## 1. Core concepts

### Network applications and sockets

**Network applications** exchange data over a network. Most use **TCP/IP**. A **socket** is the **programming interface** for this exchange. Sockets operate at **Layer 4 (Transport)** of the OSI model. The OS provides them.

- A socket is identified by **(IP address, port)**.
- A **TCP connection** involves two sockets: one on the client, one on the server (the **connection socket** returned by `accept()`).
- **Server:** binds to a port, **listens**, and **accepts** connections. **Client:** **connects** to (host, port).

In Python, use the standard module **`socket`** and create a socket with `socket.socket()`. The same call is used for both client and server; what you do next differs (bind/listen/accept vs connect).

### Ports

A **port** is a 16-bit number (0–65535) that identifies the application on a host. Use ports in **1024–65535** (e.g. 9090). The server **binds** the socket to a port with `bind()`. Always **close** sockets when done.

### TCP: how data is exchanged

**Server:** `socket()` → `bind(host, port)` → `listen()` → `accept()` (blocks until a client connects) → use the returned connection socket for `recv()` / `send()` → `close()` the connection socket.

**Client:** `socket()` → `connect(host, port)` → `send()` / `recv()` → `close()`.

`bind(('', port))` with an empty string means “all interfaces.” **accept()** and **recv()** are **blocking**. TCP provides a **byte stream**, not discrete messages. For **text**, use **encode()** before sending and **decode()** after receiving. If both sides call **recv()** first, they **deadlock**; typically the **client sends first**.

### Improvements

- **Reusable server:** Put “accept → handle → close connection” inside a **while True** loop.
- **Avoid blocking forever:** Use **sock.settimeout(seconds)**; **accept()** and **recv()** then raise **socket.timeout**.
- **Port “already in use” after restart:** Use **SO_REUSEADDR** before **bind()**: `sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)`.
- **Sending objects:** Use **pickle.dumps(obj)** before send, **pickle.loads(bytes)** after recv. Use pickle only with **trusted** peers; for untrusted data use JSON.

### TCP vs UDP

| | TCP (SOCK_STREAM) | UDP (SOCK_DGRAM) |
|---|-------------------|------------------|
| Connection | Yes | No |
| Server | listen, accept, recv/send on conn | bind, recvfrom, sendto |
| Delivery | Reliable, in-order | No guarantee |

**Key takeaways:** (1) Server binds and calls **accept()** to get a connection socket; client calls **connect()**. (2) Data on the wire is **bytes**; for text use **encode**/**decode**. (3) **recv()** returns empty bytes when the other side closes. (4) If both sides **recv()** first, they **deadlock**; client usually sends first.

---

## 2. Practical steps

Run the **server first**, then the **client**. Port **9090** for TCP, **9091** for UDP, **9092** for bank.

| Steps | Topic | Port |
|-------|--------|------|
| 1–4 | TCP echo (one message → many → reusable → timeouts) | 9090 |
| 5 | TCP + pickle (send/receive a dictionary) | 9090 |
| 6 | UDP echo | 9091 |
| 7 | Simple bank (TCP + JSON) | 9092 |

### Step 1: One client, one message

Server accepts one connection, receives up to 1 KB, echoes it back, closes the connection, then exits. Implement in folder **01_tcp_echo_one**.

**Server:** `socket()` → `bind(('', 9090))` → `listen(1)` → `accept()` → `recv(1024)` → `send(data)` → `conn.close()`.

**Client:** `socket()` → `connect(('localhost', 9090))` → `send(b"hello")` → `recv(1024)` → `close()` → `print(data.decode())`.

### Step 2: One client, many messages (until "exit")

Server reads in a loop until the client closes or sends `"exit"`. Client sends lines until the user types `exit`. Implement in **02_tcp_echo_many**.

### Step 3: Reusable server

After closing one client, the server returns to **accept()** and waits for the next. Use the same client as Step 2. Implement in **03_tcp_echo_reusable**.

### Step 4: SO_REUSEADDR and timeouts

**SO_REUSEADDR** before bind; **settimeout(seconds)** on the listen socket (e.g. 30) and on the connection socket (e.g. 60). Handle **socket.timeout** and **KeyboardInterrupt**. Client: settimeout(10); handle timeout and ConnectionRefusedError. Implement in **04_tcp_echo_timeouts**.

### Step 5: Sending a Python dictionary with pickle

**pickle.dumps(obj)** produces bytes; **pickle.loads(bytes)** restores the object. Server receives bytes, deserializes, echoes bytes back. Implement in **05_pickle_echo**. Use pickle only with trusted peers.

### Step 6: UDP echo

UDP is **connectionless**: no listen/accept. Server **binds** and uses **recvfrom()** / **sendto(data, addr)**. Client uses **sendto(data, (host, port))** and **recvfrom()**. Use **SOCK_DGRAM** and port **9091**. Implement in **06_udp_echo**.

### Step 7: Simple bank (TCP + JSON)

Client sends JSON (e.g. `{"action": "balance"}` or `{"action": "deposit", "amount": 50}`); server replies with JSON (e.g. `{"ok": true, "balance": 100}`). Implement a **Bank** class with `balance()`, `deposit(amount)`, `withdraw(amount)`. Port **9092**. Client uses **input()** for action and amount. Implement in **07_bank_json**.

---

## Folder map — what to implement

| Folder | Task | Port |
|--------|------|------|
| **01_tcp_echo_one** | TCP echo: one connection, one message. Server echoes one `recv` back and exits. Client sends one message (e.g. `b"hello"`), prints echo. | 9090 |
| **02_tcp_echo_many** | TCP echo: one client, many messages until user types `"exit"`. Server loop: recv → echo; break when data is `"exit"` or connection closed. Client: input loop, send each line, print echo; break on `"exit"`. | 9090 |
| **03_tcp_echo_reusable** | TCP echo: server accepts many clients one after another (outer `while True`: accept → handle client → close conn → repeat). Client same as Step 2. | 9090 |
| **04_tcp_echo_timeouts** | TCP echo with SO_REUSEADDR and timeouts. Server: set SO_REUSEADDR before bind; settimeout on listen socket (e.g. 30 s) and on conn (e.g. 60 s); handle `socket.timeout` and KeyboardInterrupt. Client: settimeout (e.g. 10 s); handle timeout and ConnectionRefusedError. | 9090 |
| **05_pickle_echo** | Send a Python dict over TCP using pickle. Server: recv bytes, `pickle.loads()`, print object, send same bytes back. Client: build a dict, `pickle.dumps()`, send, recv, `pickle.loads()`, print. | 9090 |
| **06_udp_echo** | UDP echo. Server: SOCK_DGRAM, bind to 9091, loop recvfrom → sendto(data, addr). Client: SOCK_DGRAM, sendto to (localhost, 9091), recvfrom, print. | 9091 |
| **07_bank_json** | Simple bank over TCP with JSON. Server: implement a `Bank` class (balance, deposit, withdraw); bind to 9092; accept one connection; recv JSON (e.g. `{"action":"balance"}` or `{"action":"deposit","amount":50}`); call bank methods; send JSON reply (e.g. `{"ok":true,"balance":100}`). Client: use `input()` for action (balance/deposit/withdraw) and amount; send one JSON request; print balance or error. | 9092 |
| **08_assignment4_port_host** | Echo server and client with config from user. Server: ask for port (default 9090), validate 1024–65535. Client: ask for host (default localhost) and port (default 9090), then connect and echo one message. | user |
| **09_assignment5_log** | Echo server that writes service messages to a log file (e.g. `server.log`) instead of the console. One function for logging (e.g. `log(msg)`); use it instead of `print`. Use any echo client (e.g. from 01) to test. | 9090 |
| **10_assignment6_auto_port** | Echo server that tries ports 9090, 9091, … until bind succeeds; print the chosen port. Use SO_REUSEADDR. Client: ask user for the port the server printed, then connect and echo. | 9090+ |

---

## 2b. Independent assignments (4, 5, 6)

- **Assignment 4 (folder 08):** Server asks for port (default 9090); client asks for host (default localhost) and port. Validate port 1024–65535.
- **Assignment 5 (folder 09):** Server writes service messages to a log file (e.g. `server.log`) instead of the console. Use a `log(msg)` function.
- **Assignment 6 (folder 10):** Server tries ports 9090, 9091, … until bind succeeds; print the port. Use SO_REUSEADDR. Client asks for the port and connects.

---

## 2c. Complete the code (exercises with hints)

Use the concepts from Section 1. Fill in the blanks; check hints before suggested answers.

- **Exercise A — Echo client (TCP):** Create socket, **connect** to ('localhost', 9090), **send** bytes, **recv(1024)**, **close**, **decode** for print. (Answers: `socket.socket()`, `connect`, `send`, `recv(1024)`, `close()`, `decode()`.)
- **Exercise B — Reusable server:** Outer **while True:**, **accept()**, handle client, **conn.close()**.
- **Exercise C — UDP client:** **SOCK_DGRAM**, **sendto**(data, ('localhost', 9091)), **recvfrom(1024)**.
- **Exercise D — Encode/decode:** **send**(msg.**encode()**), **decode()** for received bytes.

---

## 3. Control questions and answers

1. **How do client and server sockets differ?** Server: bind, listen, accept (gets a new socket per client for recv/send). Client: connect, then send/recv on the same socket.
2. **How can you transfer text through sockets?** Sockets carry bytes. Use **encode()** before sending and **decode()** after receiving; same encoding (e.g. UTF-8) on both sides.
3. **Which socket operations block?** **accept()**, **recv()**, **connect()**; **send()** can block if the buffer is full. Use **settimeout(seconds)** to limit waiting.
4. **What are the main differences between TCP and UDP?** TCP: connection, reliable, in-order. UDP: no connection, recvfrom/sendto, no delivery guarantee.
5. **Which calls are server-only?** **bind()**, **listen()**, **accept()**. The client uses **connect()** and does not call bind, listen, or accept.
6. **At what OSI layer do sockets work?** **Layer 4 — Transport layer** (TCP/UDP).

---

## 4. Full list of assignments for independent work

| # | Assignment |
|---|------------|
| 1 | Check connection from local, virtual, and remote machine (bind all interfaces; use server IP for remote client). |
| 2 | Server reads in a loop until client sends "exit". |
| 3 | Server keeps listening after a client disconnects (reusable). |
| 4 | Port and host from user input; safe defaults and validation. |
| 5 | Server messages to a log file. |
| 6 | Server auto-selects port if in use; print the port. |
| 7 | Identification server: recognize client by IP; greet by name or ask and store. |
| 8 | Authentication: username/password, secure storage, session token. |
| 9 | Helper functions: send/receive text with length-prefix. |
| 10 | Multiple messages in turn (Step 2/3 style or length-prefixed). |
| 11 | Multi-user chat (e.g. UDP broadcast). |

---

## 5. Quick reference

| Concept | Brief |
|--------|--------|
| **TCP server** | `socket()` → `bind()` → `listen()` → `accept()` → `recv()`/`send()` on conn → `conn.close()`; reusable: loop from `accept()`. |
| **TCP client** | `socket()` → `connect()` → `send()`/`recv()` → `close()`. |
| **Text** | **encode()** before send, **decode()** after recv. |
| **Pickle** | **pickle.dumps(obj)** / **pickle.loads(bytes)**. Use only with trusted peers. |
| **UDP** | **SOCK_DGRAM**; **recvfrom()** / **sendto(data, addr)**. No connection. |
| **SO_REUSEADDR** | Before `bind()` so the port can be reused soon after close. |
| **settimeout** | Limits blocking; **accept()** and **recv()** raise **socket.timeout** after the given seconds. |

---

## Running your code

In each folder, from the project root:

```bash
cd 01_tcp_echo_one
python server.py
# In another terminal:
python client.py
```

Then move to `02_tcp_echo_many`, and so on. Always start the **server** before the **client**.

---

## Before you submit

- [ ] All folders `01_`–`10_` contain your implemented **source code**.
- [ ] The **`results/`** folder contains run outputs (or screenshots) for at least steps 01, 02, 06, and 07.
- [ ] **`STUDENT_INFO.txt`** is filled with your **name** and **group** (or student ID).
- [ ] The whole lab (code + results + README + STUDENT_INFO) is pushed to **your own remote repo**.
- [ ] You have submitted your **repo URL** to your instructor.
