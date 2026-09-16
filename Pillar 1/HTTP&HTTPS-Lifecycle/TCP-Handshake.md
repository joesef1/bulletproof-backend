# TCP Handshake: Shaking Hands Before Any Data Flows

## The One-Sentence Summary

**Before a browser and a server can exchange a single byte of data, they perform a 3-message ceremony called the TCP three-way handshake (`SYN` → `SYN-ACK` → `ACK`) to agree that they're both alive, ready, and know each other's sequence numbers.**

Everything HTTP sends — every webpage, every API response — travels over a TCP connection that started with exactly this 3-packet routine.

---

## TCP à la Mode

### The One-Sentence Summary

**TCP (Transmission Control Protocol) is the "checked and reliable courier" that guarantees every byte you send arrives at the destination, in order, with no duplicates — by breaking data into numbered segments and re-sending anything that gets lost.**

If DNS answers _"where is the server?"_, and IP answers _"how do I route packets across networks?"_, then TCP answers \*"how do I make sure the conversation is complete and in order?"\_

---

## Why the Handshake Exists (The Problem)

The internet is fundamentally unreliable. Packets can be **lost, duplicated, reordered, or dropped** at any moment. Two computers that have never talked before can't just start blasting data at each other — they need to:

1. **Prove they're both there** (the other side is alive and listening).
2. **Agree on sequence numbers** so they can detect lost/reordered data later.
3. **Agree on parameters** (window sizes, options) for the connection.

That agreement is a **handshake** — exactly like two people meeting and shaking hands before having a real conversation: it's a polite, explicit "I see you, you see me, let's talk."

> **The phone analogy:** You don't just start talking when someone answers. First there's the handshake of the connection itself — the phone rings, someone says "Hello?", you say "Hi, it's me." Only _then_ does the actual conversation start. TCP does the same on every connection, forever, billions of times a day.

---

## The Three Messages: The Handshake Itself

Here's the famous diagram — the single most important picture in all of networking:

![TCP Three-Way Handshake — diagram by Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/7/71/TCP_Three-Way_Handshake.svg)

_Diagram source: [Wikimedia Commons — File:TCP Three-Way Handshake.svg](https://commons.wikimedia.org/wiki/File:TCP_Three-Way_Handshake.svg)_

Now let's decode it in the simplest possible way. Remember the three letters:

```
 CLIENT                                        SERVER
   │                                              │
   │  1.  SYN  "I want to talk. My num is X"      │
   │ ───────────────────────────────────────────►  │
   │                                              │   <-- server
   │  2.  SYN-ACK  "I got you.                    │       knows:
   │              Your num + 1 confirmed.         │       client can SEND
   │              My num is Y."                   │
   │ ◄─────────────────────────────────────────── │
   │                                              │
   │  3.  ACK  "Got your Y, we're synced.         │   <-- now client
   │            GO!"                              │       knows:
   │ ───────────────────────────────────────────►  │       server can SEND
   │                                              │       AND RECEIVE
   │ ◄══════ ESTABLISHED — data can now flow ═══► │
```

### Step by step, in plain words

| Step | Packet                                  | Who sends it | Plain English translation                                                         |
| ---- | --------------------------------------- | ------------ | --------------------------------------------------------------------------------- |
| 1    | **SYN** (Synchronize)                   | Client       | _"Hi server, I'm here. I'd like to talk. My starting sequence number is X."_      |
| 2    | **SYN-ACK** (Synchronize + Acknowledge) | Server       | _"Hi client, yes I heard you — I confirm your X+1. My own starting number is Y."_ |
| 3    | **ACK** (Acknowledge)                   | Client       | _"I got your Y. Everything is confirmed. Let's GO."_                              |

The name **SYN-ACK** is one flag packet that does double duty: it acknowledges the client's SYN **and** carries the server's own SYN. That's why 3 packets do the job that "conceptually" wants 4 (two SYN + two ACK).

---

## Why 3, and Not 2?

This is the #1 interview question about the handshake. Think about what each side learns:

- After **1 packet** (SYN): the server knows the _client_ can send.
- After **2 packets** (SYN + SYN-ACK): the _client_ knows the server is there and can both send and receive. But the **server still doesn't know** whether the client can receive.
- After **3 packets** (SYN + SYN-ACK + ACK): the server learns the client can receive. Now **both sides** have proof that the connection works **in both directions**.

So 2 messages is not enough — one side is left guessing. And 4 would be wasteful, because the server's SYN and ACK can be combined into one packet.

> **The proof-of-both-directions idea:** A handshake is a _bidirectional liveness test_. "Send, receive, send-receive" — each packet proves half a direction. Three is the smallest number that proves every direction both ways.

---

## Sequence Numbers: Why the Handshake Is Actually Useful

The handshake isn't just politeness — it exchanges **Initial Sequence Numbers (ISN)**. Every byte of data gets a sequence number, so the receiver can:

- detect a **missing** byte and ask for a re-send,
- detect a **duplicate** and discard it,
- detect **out-of-order** arrival and reorder it.

The handshake sets the _starting point_ for both directions:

```
Client:  "my bytes start at 100"   (ISN = 100)
Server:  "my bytes start at 500"   (ISN = 500)

After handshake:
  - Client's 1st real byte = 101
  - Server's 1st real byte = 501
Both sides now know what to expect. The ACK numbers in the handshake
(each side sends the other's ISN + 1) prove the ISN was received.
```

Importantly, ISNs are **random** on modern systems — mostly to prevent an attacker from _guessing_ sequence numbers and injecting fake packets into a connection (connection hijacking).

---

## Backend Developers' Must-Know Facts

You will never write the handshake yourself — but it decides your latency, your tools, and your debugging.

### 1. Every connection costs 1.5 round-trips (RTT) — before any data

Drawing the timeline makes it hurt:

```
 t=0ms     Client ── SYN ───────────►  (half RTT)
 t=28ms   Client ◄────────── SYN-ACK ──  (half RTT)
 t=56ms   Client ── ACK + HTTP GET ─►  (connection usable at 56ms!)
```

The handshake adds **1.5 RTT** (roughly one full round trip) to _every new connection_. On **HTTPS** there's ALSO a TLS handshake on top (your next topic), so the total cost before the first HTTP byte is ~**2-3 RTT** on TLS 1.3.

**Why it matters:** this is exactly why your tools/preloaders/HTTP clients **reuse connections** (keep-alive) instead of opening a new one per request.

### 2. HTTP/1.1 keep-alive exists because of this handshake

- **HTTP/1.0** opened a fresh TCP connection per request → handshake cost paid on _every_ request.
- **HTTP/1.1** defaults to **keep-alive** → one handshake, many requests on the same connection. The O'Reilly High Performance Browser Networking classic: fetching a 64KB file on a _new_ connection took **264ms**; on an _existing_ connection the same file took **~80ms shorter** — all because the handshake was skipped.

### 3. SYN flood attacks target this exact ceremony

A DoS attack: the attacker sends thousands of SYN packets but **never completes step 3** (no ACK). The server dutifully allocates memory/resources for each half-open connection until it runs out and can't serve real clients. Defenses: **SYN cookies** (don't reserve state until the connection proves itself) and rate-limiting firewalls.

### 4. Port scanning uses the handshake — or its absence

`nmap -sS` (SYN scan) sends a SYN and watches what comes back:

- **SYN-ACK** → port is **open** (listening).
- **RST** → port is closed.
- **No response / filtered** → firewall blocked it.

This is exactly why you lock down your cloud security groups — an unexposed port is invisible to the handshake.

### 5. `netstat` / `ss` states you'll actually see

```
LISTEN          server waiting for SYNs
SYN_SENT        client sent SYN, waiting for SYN-ACK
SYN_RECV        server got SYN, sent SYN-ACK, waiting for ACK
ESTABLISHED     the handshake is done — data can flow
TIME_WAIT       connection closed, timer running to avoid stale packets
```

If you see thousands of `SYN_RECV` in production with high load — suspect a **SYN flood** or misconfigured listener.

---

## How to See It Live (Do This Yourself)

```bash
# 1. Watch packets live with tcpdump (needs sudo):
sudo tcpdump -i en0 'tcp port 443 and host example.com'

# 2. See DNS → TCP → TLS in one shot:
curl -v https://example.com 2>&1 | grep -E "Connected|SSL|handshake|transferring"

# 3. In Chrome DevTools → Network → click a request → "Timing" tab:
#    "Initial connection" = the TCP handshake RTT
#    "SSL"               = the TLS handshake (next topic!)
```

The three-packet signature you're looking for in `tcpdump`:

```
1  client → server   Flags [S]                                seq=...
2  server → client   Flags [S.]   (SYN + ACK set)             seq=...
3  client → server   Flags [.]    (ACK set)                   ack=...
```

---

## How to Memorize It

### The flags — **"S-S-A"** or **"SYN, SYN-ACK, ACK"**

> Think: **S**ay hi → **S**ay hi back → **A**gree.
>
> Or the classic: **"Gimme five, got it, done."**
>
> - SYN = _gimme five_ (I want to connect)
> - SYN-ACK = _got it_ (yes, and I'm offering my hand too)
> - ACK = _done_ (shake complete)

### Why three (not two) — **"We both have to prove both directions"**

> MEMORY HOOK: 3 packets prove "you hear me, I hear you, we both hear each other."
> Say it as a triangle: **SEND → RECEIVE → CONFIRM.**

### Nearby numbers — the mnemonic **"Plus One"**

> Every ACK says the _other side's_ ISN **+ 1**. If you remember nothing else: **ack = their number + 1.**
>
> - Server ACKs: `client_ISN + 1`
> - Client ACKs: `server_ISN + 1`

### The whole handshake in one sentence

> **"Client SYN (pick a number) → Server SYN-ACK (confirm yours + 1, here's mine) → Client ACK (confirm mine + 1) → ESTABLISHED."**

---

## Where This Fits in the Bigger Picture

At this point you've stacked the layers. Look at how your "simple" request is built:

```
┌───────────────────────────────────────────────────────────────┐
│  HTTP request   (the actual GET/POST — the "what" you want)   │
├───────────────────────────────────────────────────────────────┤
│  TLS handshake  (encryption — the next lesson)                │
├───────────────────────────────────────────────────────────────┤
│  TCP handshake  (the 3-way ceremony — THIS lesson)            │
├───────────────────────────────────────────────────────────────┤
│  IP             (routing — which computer, between networks)   │
├───────────────────────────────────────────────────────────────┤
│  DNS            (resolving the hostname → IP — the first step) │
└───────────────────────────────────────────────────────────────┘
```

Every HTTPS request pays for: DNS → TCP handshake → TLS handshake → then HTTP data. That's _why_ sites feel slow on the first visit and fast on repeat visits (caching + connection reuse).

---

## Quick Self-Check (Interview-Style Questions)

1. What are the exact 3 packets of the TCP handshake, and who sends each one?
2. Why can't the handshake be done with just 2 messages?
3. What does "ISN" stand for, why is it random, and why does each ACK add 1?
4. What happens to HTTP/1.0 vs HTTP/1.1 regarding connections, and why does it relate to this handshake?
5. What's a SYN flood, and how do SYN cookies defend against it?

<details>
<summary>Answers (try first, then peek)</summary>

1. **Client → SYN**, **Server → SYN-ACK**, **Client → ACK**.
2. Because both directions must be proven usable. After 2 packets, the server still doesn't know the client can _receive_ — the client's final ACK closes that loop. And the server's SYN+ACK can be combined, so 3 is minimal.
3. **Initial Sequence Number** — the random starting number for each direction's byte counting. It's random so attackers can't guess it and inject packets. ACK = the other side's ISN **+ 1**, proving you received their ISN.
4. HTTP/1.0 opened a new connection (and thus a fresh handshake) per request — expensive. HTTP/1.1 introduced **keep-alive / persistent connections**, reusing one TCP connection (one handshake) for many requests.
5. Attackers send floods of SYNs without finishing the ACK, exhausting server resources on half-open connections. **SYN cookies** statelessly encode the connection info in the SYN-ACK so the server reserves memory only once the client proves itself with the ACK.
</details>
