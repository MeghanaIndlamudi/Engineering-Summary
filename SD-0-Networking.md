# Networking Essentials — A Beginner's Lesson for System Design Interviews

> A guided, plain-English walkthrough of the "Networking Essentials" core concept from Hello Interview's *System Design in a Hurry*. Section structure mirrors the source page; explanations, analogies, diagrams, and examples are written from the ground up for someone brand-new to system design.

**Source page:** https://www.hellointerview.com/learn/system-design/core-concepts/networking-essentials

---

## How to read this lesson

Hi! I'm going to walk you through networking the way I wish someone had walked me through it before my first system design interview. Every time we hit a technical term, I'll define it in one plain sentence before using it. Every concept gets at least one real company example (Uber, Netflix, WhatsApp, etc.) so you can see *why* engineers actually choose these tools. We'll move in the exact same order the source page uses, so if you read along you'll find the same headings in the same sequence.

Here are the inline tags you'll see throughout:

- **[ANALOGY]** — an everyday comparison to make an abstract idea click.
- **[REAL-WORLD EXAMPLE]** — a named company, what they use, and *why*.
- **[INTERVIEW TIP]** — a thing that scores points in a FAANG system design interview.
- **[COMMON MISTAKE]** — a frequent wrong answer and the correct framing.
- **[DEEPER DIVE]** — optional advanced context to push your understanding further.
- **In Simple Terms** — a one-line summary at the end of each idea.

Let's go.

---

## Why networking matters at all

Before we touch the source's first heading, one quick framing point. In system design interviews, you are almost always designing a system made up of multiple computers ("servers") that have to talk to each other and to users. Every line you draw between two boxes on a whiteboard is a *network call*. Networking is the physics of those lines. If you don't know what the lines actually do, you can't reason about latency, reliability, scaling, or failure — which is most of what the interview is about.

A quick note from the source worth repeating: networking matters more in **infrastructure / distributed systems** interviews than in **full-stack / product-focused** ones. But even product-focused interviewers will probe networking when they smell weakness. So: surface-level understanding is the floor, not the ceiling.

---

## Networking 101

### What does "networking" actually mean?

At its absolute simplest: networking is how two computers send bits to each other. One device wants to ask another device for something ("hey, give me this webpage"), or push something to it ("hey, store this photo"). Networking is the entire stack of rules and machinery that makes that possible.

Why is that hard? Because the two computers can be on opposite sides of the planet, connected via dozens of intermediate machines (routers, switches, firewalls, undersea fiber cables), and the underlying medium — electrical pulses, light, radio — has no idea what a "webpage" is. So we build *layers* of abstraction. Each layer hides the messy details of the layer below it and exposes a cleaner interface to the layer above.

**[ANALOGY]** Think about ordering a pizza for delivery. *You* don't care what road the driver takes, what gas station he stopped at, or what the engine is doing. You just want the pizza on your doorstep. The pizza shop doesn't care about the chemistry of cheese melting in the oven — they trust the oven. Networking is layered like that: each layer trusts the one underneath it to "just work."

This layered model has a formal name in textbooks: the **OSI model** (Open Systems Interconnection). It has seven layers. You don't need to memorize all seven for a FAANG system design interview. You need to know three of them really well, and the source page is going to focus on exactly those three.

**In Simple Terms:** Networking is layered so that, as an app developer, you can say "send this data" without knowing how electrons get pushed down a wire.

---

### Networking Layers

The source highlights the three layers that come up over and over in system design interviews. Let's go through each one.

```
+----------------------------------------+
| Layer 7 — Application                  |  HTTP, DNS, WebSockets, WebRTC
+----------------------------------------+
| Layer 4 — Transport                    |  TCP, UDP, QUIC
+----------------------------------------+
| Layer 3 — Network                      |  IP
+----------------------------------------+
| (Layers 1–2: physical wires/wifi,      |
|  ethernet — ignored for interviews)    |
+----------------------------------------+
```

Read the diagram top-down as "closest to your app code" → "closest to the wire."

#### Network Layer (Layer 3) — IP

**Plain English:** This layer's job is *addressing and routing*. It answers the question "which machine in the world should this packet end up at, and how does it get there?" The dominant protocol here is **IP** (Internet Protocol). IP assigns each machine a numeric address (like `142.250.190.46`) and breaks your data into small chunks called **packets**. It does *not* promise that packets will arrive, or arrive in order — it just gives best-effort delivery.

**[ANALOGY]** IP is like the postal system's address-and-routing layer. You write `1600 Pennsylvania Ave NW, Washington DC` on an envelope. The postal service knows how to move it from your local post office, through sorting facilities, to the destination. It does *not* guarantee that the letter arrives, or that two letters arrive in the order you mailed them.

**[REAL-WORLD EXAMPLE]** When you load instagram.com, dozens of routers between you and Meta's data center each look at the destination IP on every packet and forward it one hop closer to its target. None of them know that you're loading a feed — they just see "send this to `157.240.x.x`."

**[DEEPER DIVE]** There's another L3 protocol called **InfiniBand** that's used inside huge ML training clusters (e.g., training GPT-class models) where you need very low latency between GPUs. Not interview material, but worth knowing it exists if an interviewer goes there.

**In Simple Terms:** Layer 3 is the world's mailroom — it knows where every machine lives and how to forward a packet toward it.

#### Transport Layer (Layer 4) — TCP, UDP, QUIC

**Plain English:** Layer 4 sits on top of IP and adds *end-to-end* features your app actually wants: reliability (did my message arrive?), ordering (did messages arrive in the right order?), flow control (am I sending too fast for the receiver?). The two protocols you must know cold are **TCP** and **UDP**, plus a modern challenger called **QUIC**.

**[ANALOGY]** If IP is the postal system, TCP is *certified mail with delivery confirmation, tracking numbers, and re-shipment if lost*. UDP is *dropping a flyer through someone's mail slot* — fast, no confirmation, no retries. QUIC is *a modern courier service that combines the reliability of certified mail with the speed of a flyer drop.*

**In Simple Terms:** Layer 4 turns "best-effort packet delivery" into something your app can rely on (TCP), or keeps it fast and lossy when that's better (UDP).

#### Application Layer (Layer 7) — HTTP, DNS, WebSockets, WebRTC

**Plain English:** This is the layer your code touches directly. It defines *what* you're saying, not *how* it gets delivered. HTTP defines requests and responses for the web. DNS defines how to translate domain names into IPs. WebSockets define persistent bidirectional channels. WebRTC defines peer-to-peer audio/video.

**[ANALOGY]** If transport is the certified-mail envelope, application protocols are the *forms inside the envelope*. The mail system doesn't know if you're sending tax paperwork or a love letter — it just delivers the envelope. Same with TCP: it doesn't know if you're sending HTTP, an email, or a Zoom signaling message.

**[INTERVIEW TIP]** When an interviewer asks "what protocol would you use for X?", they almost always mean **Layer 7**. The right answer follows from how chatty the communication is, who initiates it, and whether browsers need to talk to it. We'll spend most of this lesson on Layer 7.

**In Simple Terms:** Layer 7 is the language your app speaks; Layers 3 and 4 are the highways that carry it.

---

### Example: A Simple Web Request

Let's stitch the three layers together with the source's worked example: what happens when you type `hellointerview.com` into a browser and hit Enter.

```
You (browser)                                    hellointerview.com server
     |                                                              |
     | 1. DNS lookup: "what IP is hellointerview.com?"              |
     |------------> DNS resolver -------------> answer: 32.42.52.62 |
     |                                                              |
     | 2. TCP three-way handshake                                   |
     |    SYN     ------------------------------------------------> |
     |    <------------------------------------------------ SYN-ACK |
     |    ACK     ------------------------------------------------> |
     |                                                              |
     | 3. HTTP GET /                                                |
     |    --------------------------------------------------------> |
     |                                                              |
     | 4. (server processes the request)                            |
     |                                                              |
     | 5. HTTP 200 OK + HTML body                                   |
     |    <-------------------------------------------------------- |
     |                                                              |
     | 6. TCP four-way teardown (FIN/ACK/FIN/ACK)                   |
     |                                                              |
```

Step-by-step:

1. **DNS resolution.** Your browser asks a DNS server (Domain Name System — the internet's phonebook) "what IP address is `hellointerview.com`?" It gets back something like `32.42.52.62`.
2. **TCP handshake** — a 3-step pleasantry before any data flows:
   - **SYN** ("hello, want to talk?") from client to server.
   - **SYN-ACK** ("yes, let's talk") from server to client.
   - **ACK** ("great, we're connected") from client to server.
3. **HTTP request.** Now that there's a reliable pipe between client and server, the browser sends `GET /` over it.
4. **Server processing.** The server does the real work — looks things up in a database, renders a page, etc. *This is the only step most developers think about, but it's only one slice of total latency.*
5. **HTTP response.** Server sends back the HTML page.
6. **TCP teardown** — a 4-step goodbye (FIN, ACK, FIN, ACK) to close the connection cleanly.

**[REAL-WORLD EXAMPLE]** Every time you open a fresh Netflix.com session, this whole dance happens. Netflix has invested enormous engineering effort in shrinking each piece: their DNS is regionally cached, their TCP handshakes are kept warm with HTTP/2 connection reuse, and their content is served from Open Connect CDN nodes inside your ISP so the HTTP response itself never has to leave your city.

**[INTERVIEW TIP]** "What happens when you type a URL and press enter?" used to be the classic interview opener. It's less common at BigTech now, but knowing the answer makes you sound senior. The three takeaways the source wants you to leave with are:

1. **Abstraction is your friend.** As an app developer, you say `fetch(url)` and trust everything underneath.
2. **One "request" is actually many packets.** Each round trip costs latency. The deeper you go in the stack, the more processing happens. *This is why a load balancer that operates at Layer 7 is slower than one at Layer 4.*
3. **A TCP connection is *state* on both sides.** If you don't reuse it (via HTTP keep-alive or HTTP/2 multiplexing), you pay the handshake cost on every request.

**[COMMON MISTAKE]** Candidates often forget that DNS lookups themselves cost time and can fail. Saying "I'll just call the server" without acknowledging DNS, TCP setup, and TLS (for HTTPS) makes you sound like you've never debugged a real network.

**In Simple Terms:** One web request is: find the address (DNS) → open a pipe (TCP) → send the question (HTTP) → server thinks → send the answer back → close the pipe.

---

## Network Layer Protocols

Now we dive into the bottom of the three layers — IP.

**Plain English:** IP's job is to give every machine on the internet an address, then move packets between addresses. When a server boots up inside a data center, it gets assigned an IP address — usually automatically, by something called a **DHCP server** (Dynamic Host Configuration Protocol — a service whose only job is to hand out IP addresses).

Two flavors of IP address matter for system design:

- **Private IPs** — addresses that only mean something inside your own network. You can pick whatever you want (`10.0.0.0/8`, `192.168.x.x`, etc.). They're not routable on the public internet.
- **Public IPs** — addresses that are globally unique and registered with a **RIR** (Regional Internet Registry — an organization that hands out ranges of IPs to companies and ISPs). These are the ones internet routers know how to route to.

**[ANALOGY]** Private IPs are like apartment numbers inside a building — useful internally, but if you mail a letter just to "Apartment 4B" with no city or street, the postal service has no idea what to do. Public IPs are like full street addresses — the system knows where they are.

**[REAL-WORLD EXAMPLE]** Anything in the `17.x.x.x` range belongs to Apple. The backbone routers of the internet are configured to know that packets destined for `17.x.x.x` should be sent toward Apple's network. Similarly, `8.8.8.8` is Google's public DNS — everybody on Earth's routing tables agree to send packets for `8.8.8.8` toward Google.

**[INTERVIEW TIP]** You almost never need to design routing in a system design interview. Mentioning DHCP and that "instances get IPs assigned at boot" is plenty. Going deeper than that is usually a waste of interview time unless you're specifically interviewing for a networking team.

**In Simple Terms:** IP gives each machine an address, and the internet's routers know how to forward packets to those addresses. That's all you really need at this layer.

---

## Transport Layer Protocols

This is where the interview-relevant choice begins. The transport layer is where two machines establish an *end-to-end conversation*, not just packet-by-packet hops. Three protocols matter: **TCP**, **UDP**, and **QUIC** (newer).

The source flags that **the real interview decision is TCP vs UDP**. QUIC is "TCP with modern upgrades" — bring it up if you're going for performance bonus points, but don't lean your design on it.

**[INTERVIEW TIP]** If you can correctly justify *one* well-placed use of UDP in a system design that's otherwise TCP, you've signaled senior-level thinking. Most candidates default to TCP everywhere and never question it.

---

### UDP: Fast but Unreliable

**Plain English:** UDP (User Datagram Protocol) is the bare-minimum transport on top of IP. You send a packet, you hope it arrives, you move on. No handshake, no acknowledgments, no retries, no ordering. The receiver only knows where the packet came from and what's inside it. That's it.

**[ANALOGY]** UDP is shouting across a noisy room. You yell "the pizza's here!" If your roommate didn't catch it, oh well — they'll figure it out when you shout the next thing, or when they smell the pizza. You don't wait for confirmation, you don't repeat yourself.

#### Key characteristics

1. **Connectionless** — no handshake; you just send.
2. **No delivery guarantee** — packets may be lost silently.
3. **No ordering** — packet #5 might arrive before packet #3.
4. **Low latency** — almost no overhead. The header is tiny (8 bytes vs. TCP's 20–60).

So why ever use it? Because for some use cases, losing a packet is **cheaper** than waiting for it to be re-sent.

**[REAL-WORLD EXAMPLE]** Uber's live driver location pings. Every 4 seconds (roughly), the driver's phone sends its GPS location to Uber's servers. If one ping is lost, the very next one (4 seconds later) supersedes it anyway. Re-transmitting the lost one would be wasted bandwidth and add latency. UDP is perfect.

**[REAL-WORLD EXAMPLE]** Voice calls on WhatsApp, FaceTime audio, and Zoom audio use UDP-based protocols. If you drop a 20-millisecond chunk of audio, you'd rather hear a tiny crackle than have the audio "pause and catch up" later. TCP would pause and catch up. UDP just keeps going.

**[REAL-WORLD EXAMPLE]** Online games like *Fortnite* and *Counter-Strike* use UDP for game state updates. A late packet about where another player was 100ms ago is useless — better to drop it and use the fresh one.

**[REAL-WORLD EXAMPLE]** DNS lookups themselves use UDP! Most DNS queries fit in a single packet, and if you don't get an answer back in a few hundred ms, you just retry. No need for a heavy TCP handshake.

#### Where UDP fits in your interview answer

UDP is right when **speed beats reliability** and **the application can tolerate loss**. Live video, live audio, gaming, telemetry, DNS. The flip side, which the source flags clearly: **browsers don't natively support UDP** outside of WebRTC. So if your design has browser clients, you'll often pair a UDP path (for native apps) with a fallback HTTP/WebSocket path (for browsers).

**[INTERVIEW TIP]** If the interview is "design Facebook Live's heart-spam reactions," you can suggest UDP for the mobile app's reaction stream and a slower batched HTTP stream for browser users — same UX, different transport. That kind of nuance scores points.

**In Simple Terms:** UDP is fast and forgetful — perfect when losing a message is cheaper than waiting for it.

---

### TCP: Reliable but with Overhead

**Plain English:** TCP (Transmission Control Protocol) is the opposite philosophy. It does whatever it takes to make sure data arrives, in order, intact. Before any data flows, the two sides do a 3-way handshake to set up a **connection** (also called a **stream**). Every packet sent gets acknowledged; if an acknowledgment doesn't come back in time, TCP re-sends. TCP also slows itself down if the network is congested, and slows the sender if the receiver is overwhelmed.

**[ANALOGY]** TCP is like a phone call with constant "uh-huh, got it" feedback. Before you talk, you dial and the other side picks up ("hello?", "hi, can you hear me?", "yes, go ahead" — that's your handshake). During the call you naturally pause when the line drops and repeat what got missed. When you're done, you both say "okay, bye!" before hanging up.

#### Key characteristics

1. **Connection-oriented** — handshake first, data second.
2. **Reliable, ordered delivery** — packets are reassembled in the order they were sent; lost packets are retransmitted.
3. **Flow control** — the receiver tells the sender how much buffer space it has, so it doesn't drown.
4. **Congestion control** — if packets are getting lost (suggesting the network is jammed), TCP slows itself down automatically.

**[REAL-WORLD EXAMPLE]** Loading the Amazon checkout page. You absolutely cannot have your credit card number arrive out of order, or with bytes missing. Every byte must match. TCP guarantees it.

**[REAL-WORLD EXAMPLE]** Every time you send a WhatsApp text message (not audio call, just a text), it travels over TCP. WhatsApp would rather make you wait 200ms than deliver a garbled message.

**[REAL-WORLD EXAMPLE]** Database replication between primary and replicas at companies like Stripe, Shopify, or Airbnb runs over TCP. Losing a single write to your replica is unacceptable; waiting an extra few ms is fine.

#### Where TCP fits

Basically *everything that isn't UDP*. The source's punchline: "TCP is ideal for applications where data integrity is critical — that is, basically everything where UDP is not a good fit."

**[DEEPER DIVE]** **QUIC** is a newer protocol built by Google and standardized in 2021. It runs *over UDP* but adds TCP-style reliability plus features like 0-RTT connection setup (you can start sending data on the first packet to a previously-seen server). HTTP/3 runs on QUIC. YouTube and Google Search both use HTTP/3 for many clients today. If an interviewer is performance-obsessed, mentioning QUIC/HTTP/3 as "a modernized TCP without the legacy baggage" is a nice flex.

**In Simple Terms:** TCP is slower than UDP but guarantees that every byte arrives, in the right order — which is what most applications need.

---

### When to Choose Each Protocol

The default in any FAANG system design interview is **TCP**. You don't even have to say "I'll use TCP" — interviewers assume it. You explicitly justify UDP only when the use case clearly demands it.

You'd reach for UDP when:

- **Low latency is critical** and a few lost packets won't ruin the experience (live video, real-time gaming, VoIP).
- **You're shipping high-volume telemetry or logs** where occasionally losing one is acceptable (think IoT sensors, metrics agents).
- **You don't need to support browsers** directly (or you have a parallel browser-friendly path).

**[INTERVIEW TIP]** Many real apps use **both**. A video conferencing app like Zoom or Google Meet uses TCP/HTTPS for "sign me in, give me the meeting metadata" and UDP/WebRTC for the actual audio/video pixels. Naming this mixed pattern in an interview is a high-signal move.

#### TCP vs UDP at a glance

```
                  UDP                        TCP
Connection:       None (just send)           Handshake first
Reliability:      Best-effort, no retries    Guaranteed, retransmits
Ordering:         None                       Strict order
Flow control:     None                       Yes
Congestion ctrl:  None                       Yes
Header size:      8 bytes (tiny)             20–60 bytes
Speed:            Faster                     Slower (overhead)
Used for:         Streaming, gaming, VoIP    Pretty much everything else
```

**[COMMON MISTAKE]** "TCP is more secure than UDP." Wrong — neither is secure on its own. Security comes from TLS, which runs on top of TCP (and in QUIC's case, is baked in). UDP can also be wrapped in DTLS for security.

**In Simple Terms:** Default to TCP. Pick UDP only when your use case can tolerate loss in exchange for raw speed.

---

## Application Layer Protocols

Now we're up to Layer 7 — the layer your app code actually writes. These protocols run on top of TCP or UDP and define *what* you're sending.

A useful bit of context from the source: most Layer 7 protocols are processed in **user space** (the normal part of your operating system that your app runs in), while TCP/UDP/IP live in **kernel space** (the privileged part). Kernel-space code is fast but rigid. User-space code is flexible but slower. That's why you can write your own Layer 7 protocol pretty easily, but you can't casually rewrite TCP.

**[ANALOGY]** Kernel space is the engine and transmission of a car — hard to change, but tuned for speed. User space is the GPS app — easy to update, but riding on top of the engine.

---

### HTTP/HTTPS: The Web's Foundation

**Plain English:** HTTP (HyperText Transfer Protocol) is the request-response protocol the web is built on. The client sends a *request*; the server sends back a *response*. That's the whole shape of it.

A defining property: HTTP is **stateless**. Each request stands alone — the server doesn't have to remember anything about your previous requests to answer this one. (Cookies and sessions are how apps fake state on top of stateless HTTP.)

**[ANALOGY]** HTTP is like ordering at a fast-food counter. Every order is a fresh transaction; the cashier doesn't have to remember your previous order to take this one. If they need context (like a loyalty card number), you hand it to them with the order.

**[INTERVIEW TIP]** Statelessness is a *feature*. It means any of your N servers can handle any request — making horizontal scaling trivial. In an interview, saying "I'll keep my HTTP API stateless so I can load-balance freely" is a clean win.

#### The four things in every HTTP message

1. **Method** — what action are you trying to perform?
2. **Status code** — only on responses; did it work?
3. **Headers** — key/value metadata about the request or response.
4. **Body** — the actual content (a JSON payload, an HTML page, an image, etc.).

#### Common methods

- **GET** — fetch a resource. Should not change anything on the server (it should be *idempotent*). No body.
- **POST** — create something, or trigger an action.
- **PUT** — replace an existing resource entirely.
- **PATCH** — partially update an existing resource.
- **DELETE** — remove a resource. Should be idempotent.

#### Common status codes (memorize these — they come up)

- **2xx — Success.**
  - `200 OK` — generic success.
  - `201 Created` — new resource was created.
- **3xx — Redirects.**
  - `301 Moved Permanently` — the resource lives at a new URL forever.
  - `302 Found` — temporarily moved.
- **4xx — Client errors.** (You asked for something wrong.)
  - `401 Unauthorized` — not logged in.
  - `403 Forbidden` — logged in but not allowed.
  - `404 Not Found` — doesn't exist.
  - `429 Too Many Requests` — slow down (rate-limited).
- **5xx — Server errors.** (The server failed.)
  - `500 Internal Server Error` — generic server crash.
  - `502 Bad Gateway` — your server's upstream dependency failed.

**[REAL-WORLD EXAMPLE]** When Instagram rate-limits you for refreshing too fast, you'll see `429 Too Many Requests` in the network tab. When a YouTube live stream's origin server can't reach the encoder, you'll see `502 Bad Gateway` from the CDN.

#### Headers — the flexible part

Headers are key/value pairs and are deliberately open-ended. The classic example is content negotiation:

- Client sends `Accept-Encoding: gzip, br` — "hey, I can decompress gzip or brotli."
- Server picks one, compresses the body with it, and responds with `Content-Encoding: br`.

This way old clients (that don't know brotli) still work, new clients get the fastest encoding, and the protocol degrades gracefully. **This pattern — open-ended key/value metadata that lets two parties negotiate without breaking older versions — is itself a great API-design lesson.**

#### HTTPS

HTTPS is just HTTP wrapped in **TLS** (Transport Layer Security — a protocol that encrypts everything between client and server and verifies the server's identity using a certificate). Two huge things it gives you:

1. **Confidentiality** — eavesdroppers on the wire (e.g., on the same coffee-shop Wi-Fi) can't read your traffic.
2. **Authenticity** — you know you're really talking to `chase.com` and not a fake server pretending to be it.

**[REAL-WORLD EXAMPLE]** Every modern major site — Google, Stripe, Facebook, your bank — requires HTTPS. Browsers actively warn users about plain HTTP now. If you're building a public website in an interview, HTTPS is the default and unstated.

**[COMMON MISTAKE]** "HTTPS means I can trust the request body." Nope. HTTPS proves the body wasn't tampered with *in transit*. It does NOT prove the body is honest. A malicious client can put `userId: "victim_id"` in the body and HTTPS will faithfully encrypt and deliver that lie. **Always validate user identity on the server from a signed token (e.g., a JWT or session cookie), never trust user-supplied IDs from the body.**

**[INTERVIEW TIP]** When you describe an auth flow, say "the server resolves the user identity from the session token, not from any field in the request body." Interviewers love this.

**In Simple Terms:** HTTP is request-and-response over TCP, with methods, status codes, headers, and a body. HTTPS adds encryption and server-identity proof.

---

### REST: Simple and Flexible

**Plain English:** REST (REpresentational State Transfer) is a style for building APIs *on top of* HTTP. Its core idea: think of your system as a collection of **resources** (users, posts, orders, rides), and use HTTP methods to operate on them.

**[ANALOGY]** REST treats your API like a filing cabinet. You don't say "do operation X" — you say "fetch the folder at `/users/42`" or "replace the folder at `/users/42` with these new contents" or "create a new folder under `/users`." Verbs and paths are standardized; only the contents change.

#### What a RESTful API looks like

```
GET    /users/42                  → fetch user 42
POST   /users         { ... }     → create a new user
PUT    /users/42      { ... }     → replace user 42's data
PATCH  /users/42      { ... }     → partially update user 42
DELETE /users/42                  → delete user 42

GET    /users/42/posts            → list user 42's posts
GET    /users/42/posts/7          → fetch post 7 under user 42
```

Notice three things:

1. **The path identifies a resource** (`/users/42`).
2. **The method describes the action** (GET, POST, etc.).
3. **The body carries the data** (usually JSON).

#### How to think in REST

The most common mistake new engineers make is thinking in *verbs* ("updateUser", "startGame", "sendMessage"). REST asks you to translate those verbs into *resources + methods*:

- `updateUser(id, data)` → `PUT /users/{id}` with body `data`
- `startGame(id)` → `PATCH /games/{id}` with body `{"status": "started"}`
- `sendMessage(from, to, text)` → `POST /messages` with body `{from, to, text}`

**[INTERVIEW TIP]** If you've used the source's Delivery Framework, your **Core Entities** map almost one-to-one to your REST resources. So defining entities up front gives you your API design "for free."

#### Where to use REST

REST is the right default for almost any public-facing API. It's well-understood, easy to test (a `curl` command works), and any developer in any language can call it. JSON-over-HTTP isn't the fastest format on Earth — gRPC will beat it on raw throughput — but for most workloads the bottleneck is somewhere else (database, business logic, network round-trip).

**[REAL-WORLD EXAMPLE]** Stripe's API, GitHub's public API, Twilio's API, Shopify's storefront API — all REST. ElasticSearch, called out by the source, uses REST for its admin and query interfaces.

**[INTERVIEW TIP]** In an interview, default to REST for any public-facing API. Reach for GraphQL or gRPC only when the use case justifies it — and we'll cover when that is below.

**[COMMON MISTAKE]** Naming your endpoints with verbs: `POST /createUser`, `POST /deleteUser`. This is RPC-style, not REST. Interviewers will (politely) push back. Use nouns and the right HTTP method instead.

**In Simple Terms:** REST = HTTP methods + resource URLs + JSON bodies. It's the default API style for almost everything that talks to the public internet.

---

### GraphQL: Flexible Data Fetching

**Plain English:** GraphQL is a query language for APIs, invented by Facebook and open-sourced in 2015. Instead of the server defining endpoints and the client taking whatever those endpoints return, the **client describes exactly what data it wants**, and the server returns that — no more, no less.

#### The problem GraphQL solves

In a REST world, building a complex screen often forces you into one of three ugly options:

1. **Under-fetching** — making lots of small REST calls (list of users, then 10 calls for their profiles, then 10 calls for their groups…). Each round trip adds latency.
2. **Over-fetching** — the server stuffs a giant blob into the response "just in case." Wasted bandwidth, slow parsing, and you still might not have what you need.
3. **Bespoke endpoints** — creating a custom REST endpoint per screen (`GET /users-with-profiles-and-groups`). Maintenance nightmare; every new screen needs new backend work.

**[ANALOGY]** REST is like a fixed-menu restaurant — you order Combo #3, and you get whatever they decided #3 contains. GraphQL is like a build-your-own bowl — you list exactly which ingredients you want, and that's what you get.

#### What a GraphQL query looks like

```
query {
  users(limit: 10) {
    id
    username
    profile { fullName avatar }
    groups {
      name
      category { name icon }
    }
  }
}
```

You can see the shape: pick the fields you want, follow nested relationships, get exactly that shape back as JSON. The frontend can ship a new screen without asking the backend team for a new endpoint.

#### Where to use GraphQL

GraphQL shines when:

- The frontend iterates fast and product needs change often.
- Multiple frontend clients (web, iOS, Android) need slightly different shapes of the same data.
- You have a federated schema spanning many backend teams.

**[REAL-WORLD EXAMPLE]** Facebook uses GraphQL extensively for its mobile apps — exactly the use case it was built for. GitHub exposes a GraphQL API alongside its REST API; you can query the entire issue/PR graph in one request. Shopify's Storefront API is GraphQL.

#### When NOT to reach for GraphQL in an interview

The source is clear-eyed about this. In a system design interview, your requirements are **fixed** (the interviewer just told you what the app does), so GraphQL's "rapid frontend iteration" advantage doesn't help much. Worse, the way GraphQL gets executed can hide the very database access patterns the interviewer wants to discuss.

**[INTERVIEW TIP]** Bring up GraphQL only if the interviewer explicitly mentions frontend flexibility ("we want to iterate on the UI quickly") or vague/changing requirements. Otherwise, default to REST.

**[DEEPER DIVE]** A classic GraphQL pitfall is the **N+1 problem**: a single query that looks innocent expands into hundreds of database hits on the backend. The standard fix is a tool like **DataLoader** that batches and caches resolver calls. Mention this if a GraphQL discussion deepens — it shows you've actually built one.

**In Simple Terms:** GraphQL lets the client ask for exactly the data shape it needs. Powerful for fast-moving frontends; usually overkill in an interview.

---

### gRPC: Efficient Service Communication

**Plain English:** gRPC (the "g" is Google) is a high-performance way for *services* to talk to each other. It runs over HTTP/2 and uses **Protocol Buffers** (Protobuf) — a compact binary format with a strict schema — instead of JSON.

**[ANALOGY]** REST/JSON is like writing a letter in handwritten English — humans can read it, but the envelope is big and someone still has to *parse* it. gRPC/Protobuf is like sending a coded telegram — much shorter, much faster to decode, but you need the codebook (the `.proto` file) on both ends.

#### What a Protobuf schema looks like

```
message User {
  string id   = 1;
  string name = 2;
}

service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}
```

You compile this `.proto` file with a code generator, and you get a typed client and a typed server stub in Go, Java, Python, etc. Calling `UserService.GetUser(...)` looks like a normal function call in your code — gRPC handles the network in the background.

#### Why gRPC is fast

The same `{"id":"123","name":"john doe"}` JSON (~40 bytes) becomes a binary message of ~15 bytes in Protobuf. Less data on the wire, faster parsing, no need to ship a schema in the payload. Some benchmarks show **10× higher throughput** vs REST/JSON for the same workload. gRPC also gives you streaming RPCs, deadlines, and built-in client-side load balancing.

#### Where to use gRPC

Best fit: **internal service-to-service communication** in a microservices architecture, especially when latency or throughput matters.

**[REAL-WORLD EXAMPLE]** Google uses gRPC (and its internal predecessor, Stubby) between practically every internal service. Netflix uses gRPC heavily for backend-to-backend traffic. Spotify uses it in its microservices mesh. None of them expose gRPC publicly — that's the key.

#### Why NOT for public APIs

gRPC is a binary protocol. It's awkward to call from a browser (you need a translation layer called gRPC-Web). The tooling is less mature than `curl`-and-JSON. Third-party developers don't want to deal with `.proto` files. So almost everyone runs:

```
External clients (browsers, mobile, third parties)
            |  REST / JSON
            v
       +----------+
       |  Public  |
       |   API    |
       +----------+
            |  gRPC / Protobuf
            v
       +-----------+
       | Internal  |
       | services  |
       +-----------+
```

**[INTERVIEW TIP]** "REST on the edge, gRPC inside" is a standard pattern at modern tech companies. Saying it in an interview shows you understand the trade-off. But: don't get hung up on RPC choice early in a design. Bigger wins are usually elsewhere — caching, database choice, sharding.

**[COMMON MISTAKE]** Bragging about gRPC for a system whose bottleneck is clearly the database. Optimizing serialization when your real problem is a missing index makes you look junior. *Premature optimization is the root of all evil.*

**In Simple Terms:** gRPC is fast, schema-driven, binary RPC. Use it between your internal services; keep REST for the public-facing edge.

---

### Server-Sent Events (SSE): Real-Time Push Communication

**Plain English:** SSE is a way for a *server* to push a stream of messages to a *client* over a single HTTP connection. Normally HTTP is "you ask, I answer, we're done." SSE keeps the response open and lets the server keep streaming chunks down it.

**[ANALOGY]** SSE is like subscribing to a podcast feed where the host can keep adding episodes to the same RSS feed. You opened the feed once, and new episodes appear automatically — no need to re-subscribe.

#### Why it exists

A classic JSON response delivers everything at once after the server is done:

```
{
  "events": [event1, event2, ..., event100]
}
```

You have to wait for the *whole thing* before you can do anything. Useless for live updates.

SSE instead streams each event as a separate chunk on the same response:

```
data: {"id": 1, "msg": "stock TSLA = 240.12"}

data: {"id": 2, "msg": "stock TSLA = 240.18"}

data: {"id": 3, "msg": "stock TSLA = 240.05"}
...
```

The browser exposes this through a simple JavaScript API called `EventSource`. Each `data:` line fires an event your code can react to.

#### How SSE handles disconnects

Long-lived HTTP connections get killed by all sorts of things: load balancers, idle timeouts, a tunnel through a corporate proxy, the user's flaky 5G. SSE has a built-in fix: when the connection drops, the browser automatically reconnects and sends a `Last-Event-ID` header telling the server "I last saw event #57; replay from #58." The server is expected to keep enough recent history around to satisfy this.

#### Where to use SSE

When you need **server → client one-way push** and don't need the client to push much back, SSE is the simplest tool for the job. It also has a huge advantage: it's just HTTP. Every firewall, proxy, and load balancer already knows how to handle it (mostly — see the gotcha below).

**[REAL-WORLD EXAMPLE]** OpenAI's ChatGPT web UI streams its token-by-token responses to your browser using SSE. Each token arrives as a separate event. WhatsApp Web uses similar streaming techniques. Online auction sites (eBay-style) often use SSE to push the latest high bid to all viewers.

**[REAL-WORLD EXAMPLE]** Stock-price tickers, sports-score updates, social-media live counts ("3,412 people watching now") — all great fits for SSE.

**[DEEPER DIVE]** SSE's nasty edge case: some misbehaving networks or proxies *buffer the entire stream* before forwarding it. The client gets nothing for a long time, then a huge dump — defeating the whole point. The standard mitigations are setting `Cache-Control: no-cache` and `X-Accel-Buffering: no`, and sending periodic keepalive comments (`: ping\n\n`). If an interviewer probes here, mentioning that you've thought about buffering proxies is a signal you've actually shipped SSE.

**In Simple Terms:** SSE is server-to-client streaming over plain HTTP. Use it when the server has frequent updates and the client mostly just listens.

---

### WebSockets: Real-Time Bidirectional Communication

**Plain English:** A WebSocket is a *persistent, full-duplex* TCP connection between client and server, available even in browsers. "Full-duplex" means both sides can send at any time, without waiting for the other to ask. Once it's open, it stays open until someone closes it.

**[ANALOGY]** HTTP is walkie-talkie: one person talks, the other listens, then they switch. WebSocket is a phone call: both ends can speak whenever they want, and the line stays open.

#### How a WebSocket connection starts

1. The client makes a normal HTTP request with an `Upgrade: websocket` header.
2. The server agrees with a `101 Switching Protocols` response.
3. From that moment on, the same TCP connection is no longer speaking HTTP — it's speaking WebSocket. Both sides can send binary or text messages at any time.
4. The connection stays open until someone explicitly closes it (or the network drops it).

```
Client                                  Server
  |--- HTTP GET / Upgrade: websocket --->|
  |<-- HTTP 101 Switching Protocols  ----|
  |                                      |
  |======== WebSocket open ==============|
  |--- "user typed: hello" ------------->|
  |<-- "friend is typing..." ------------|
  |<-- "new message from Alice"  --------|
  |--- "user sent: hi back" ------------>|
  ...                                   ...
```

#### Where to use WebSockets

WebSockets are the right tool when you need **high-frequency, persistent, bidirectional** communication:

- Multiplayer real-time games
- Chat apps (typing indicators, instant delivery)
- Collaborative editing (cursors, live edits)
- Trading dashboards with two-way actions

**[REAL-WORLD EXAMPLE]** Slack uses WebSockets to deliver messages to clients in real time and to let the client send typing indicators back. WhatsApp Web maintains a WebSocket to QR-pair you with your phone and stream messages. Google Docs uses WebSockets so every keystroke from every editor flows back and forth instantly. Robinhood and many crypto exchanges use WebSockets for live price ticks plus order entry.

**[INTERVIEW TIP]** Justify WebSockets explicitly. The bar is: *"I need bidirectional, low-latency, high-frequency messaging, and SSE-style one-way push isn't enough."* If you can't say that out loud, default to SSE or long polling. **Launching into WebSockets without justification is one of the fastest ways to lose interview points.**

**[COMMON MISTAKE]** Forgetting that WebSocket servers are **stateful**. Every open WebSocket is a TCP connection eating memory and file descriptors on a specific server. You can't load-balance them with a vanilla L7 load balancer the way you would HTTP. You need:

- Layer 4 load balancing (sticky-by-connection) — discussed below.
- Some way to route a "message for user X" to the *specific server* that's currently holding user X's WebSocket. (Usually a pub/sub layer like Redis or Kafka feeds messages to whichever server has that user connected.)

**[DEEPER DIVE]** Infrastructure between client and server (corporate proxies, ancient load balancers, some mobile carriers) sometimes silently kills idle WebSocket connections. Production WebSocket implementations send periodic *ping/pong* frames every 20–30s to keep them alive and detect dead peers.

**In Simple Terms:** WebSockets are a long-lived two-way pipe. Use them only when both sides need to push to each other in real time.

---

### WebRTC: Peer-to-Peer Communication

**Plain English:** WebRTC (Web Real-Time Communication) lets two browsers — or two mobile apps — talk **directly to each other**, without routing all the data through a central server. It's the only Layer 7 protocol the source covers that uses UDP. It's built for real-time audio, video, and arbitrary data channels.

**[ANALOGY]** Most internet apps are like everyone calling each other through a central switchboard operator (the server). WebRTC is like two people walking up and chatting face-to-face — once they find each other, the operator drops out of the conversation.

#### Why peer-to-peer is *hard*

In an ideal world, your browser would just open a TCP/UDP socket directly to your friend's browser. But there are two killer obstacles:

1. **Most clients don't accept incoming connections.** Their firewalls or operating systems block it by default — for very good security reasons.
2. **NAT** (Network Address Translation) — your home Wi-Fi router gives all your devices the same public IP and rewrites ports on the fly. Other devices on the internet have no way to address your laptop directly.

So WebRTC needs help getting two peers connected. Three pieces of supporting infrastructure exist:

- **Signaling server** — a centralized matchmaker. Both peers connect to it (usually via WebSocket or HTTPS) and exchange contact info: "here's my candidate IP/port — try me." The signaling server *never* carries the actual audio/video; it just helps the peers find each other.
- **STUN server** (Session Traversal Utilities for NAT) — a small public server whose only job is to tell you "this is the public IP and port the world sees you from." Armed with that info, peers can sometimes "hole-punch" through their NATs and connect directly.
- **TURN server** (Traversal Using Relays around NAT) — the fallback. When hole-punching fails (e.g., symmetric NATs or strict corporate firewalls), TURN relays the media through a server. It's slower and costs more, but it works.

```
              +-------------------+
              | Signaling server  |
              |   (matchmaker)    |
              +---------+---------+
                        |
        +---------------+----------------+
        |                                |
        v                                v
   +----------+                     +----------+
   |  Peer A  |                     |  Peer B  |
   |  laptop  |  ◄───────────────► |  phone   |
   +----------+   direct P2P media  +----------+
        |                                |
        |   (or via TURN relay if NAT    |
        |    punching fails)             |
        +------> TURN server <-----------+
```

#### The 4-step handshake

1. Both clients connect to the signaling server and discover each other.
2. Each client asks a STUN server "what's my public IP/port?"
3. Clients share each other's IP/port via the signaling server.
4. Clients attempt a direct peer-to-peer connection. If that fails, they fall back to relaying through a TURN server.

#### Where to use WebRTC

The safest interview answer: **video and audio calling/conferencing applications**.

**[REAL-WORLD EXAMPLE]** Google Meet and Discord voice both use WebRTC for the audio/video media streams. Facebook Messenger video calls use WebRTC under the hood. Houseparty was famously WebRTC-heavy.

**[REAL-WORLD EXAMPLE]** Some collaborative editors (like Figma for certain real-time features, or the experimental Notion peer features) use WebRTC's *DataChannel* for low-latency cursor and presence updates — though most collaboration tools (Google Docs included) use plain WebSockets because they need a central source of truth.

**[COMMON MISTAKE]** Trying to design every collaboration system as peer-to-peer with WebRTC. Most collaboration needs a central server anyway (to store the document, do auth, audit). The source's pointed warning: "I've seen more candidates go wildly off trail trying to design peer-to-peer systems using WebRTC than I have seen them successfully implement them."

**[INTERVIEW TIP]** Limit WebRTC to clearly peer-to-peer media use cases — video/audio calls — and you'll never get burned. If you can name **STUN, TURN, and a signaling server** as three distinct pieces, you've already cleared the bar most candidates set.

**[DEEPER DIVE]** Real video conferencing at scale (Google Meet with 100 participants) doesn't use full mesh peer-to-peer — N² connections would melt your laptop. Instead they use an **SFU** (Selective Forwarding Unit), a media server that receives each peer's WebRTC stream and selectively forwards copies. This is still "WebRTC" on the wire, but it's no longer pure P2P. Mention SFUs if your interviewer goes deep on conferencing.

**In Simple Terms:** WebRTC is the protocol for direct peer-to-peer audio, video, and data. Use it for calling/conferencing — leave it alone for everything else.

---

## Load Balancing

You've now seen all the protocols. Time for the *scaling* question: when one server isn't enough, what do you do?

You have two options:

- **Vertical scaling** — make the single server bigger (more CPU, more RAM). Simple but has a hard ceiling and a single point of failure.
- **Horizontal scaling** — add more servers. Much more flexible, scales nearly indefinitely, and gives you fault tolerance (if one dies, others handle the load).

```
   Vertical scaling           Horizontal scaling
   ----------------           ------------------

      +-----+                 +---+  +---+  +---+
      |     |                 |   |  |   |  |   |
      | big |   versus        +---+  +---+  +---+
      |  box|                 +---+  +---+  +---+
      |     |                 |   |  |   |  |   |
      +-----+                 +---+  +---+  +---+
```

In interviews, **horizontal scaling is the default answer**. But the moment you have N servers, you have a new question: *which server does each request go to?* That's load balancing.

**[INTERVIEW TIP]** The source's author personally prefers vertical scaling when possible because modern hardware is much beefier than people assume. But for interviews, expect to talk about horizontal scaling and load balancers.

---

### Types of Load Balancing

There are two main shapes:

1. **Client-side load balancing** — the client itself picks which server to talk to.
2. **Server-side (dedicated) load balancing** — a separate machine in front of your servers makes the choice for the client.

Both have their place.

---

#### Client-Side Load Balancing

**Plain English:** The client gets the full list of backend servers (from a "service registry" — basically a directory) and picks one itself. No middleman.

**[ANALOGY]** Client-side load balancing is like having the full menu and phone numbers of every pizza shop in town and calling whichever one is closest yourself. Server-side is like calling 311 and letting the operator decide who serves you.

**Why it's appealing:** zero extra network hops. The client → server path is direct. The client just needs to refresh its list periodically.

##### Example: Redis Cluster

In Redis Cluster (a distributed key-value store), each node knows about every other node via a gossip protocol — a peer-to-peer chat where each server tells its neighbors "I'm alive, here's who I know about." When a client first connects, it grabs the cluster topology and learns which **shard** (chunk of the keyspace) lives on which node. From then on, the client hashes the key locally to figure out the right node and sends the request directly to it.

If the client guesses wrong (because the cluster reshuffled), the Redis node politely responds with `MOVED <new-node>`, and the client updates its local map.

**[REAL-WORLD EXAMPLE]** Companies that run heavy Redis workloads (Twitter, GitHub, Pinterest) all rely on this client-side routing pattern to avoid putting an extra hop in front of their cache.

##### Example: DNS

DNS itself is a sneakier form of client-side load balancing. When you resolve `google.com`, the DNS server typically returns several IP addresses, often in a rotated order. Different clients get different orderings; clients usually try the first one. The net effect: traffic spreads across many IPs without any central load balancer doing it.

**[REAL-WORLD EXAMPLE]** This is also how you avoid your load balancer itself being a single point of failure. You stand up *two* load balancers in different regions, give them different IPs, and let DNS round-robin between them. If one goes down, clients trying the second one succeed.

##### Where it fits

Client-side works great when either:

1. **You control the clients** and can update them easily (e.g., internal microservices using gRPC, where every service has the same client library).
2. **You have lots of clients but can tolerate slow updates** (e.g., DNS, where TTLs of minutes-to-hours are the norm).

**[INTERVIEW TIP]** gRPC has built-in client-side load balancing. If you're designing internal microservices over gRPC, mentioning "the gRPC clients can do client-side load balancing against a service discovery system like Consul or etcd" is a credible, senior-sounding answer.

**[DEEPER DIVE]** The slow-update problem cuts both ways: if a server dies, clients with cached endpoints will keep hitting the dead address until their cache expires. That's why DNS TTLs are usually kept low (60s) for services that change often, and high (24h) for relatively stable infrastructure.

**In Simple Terms:** With client-side load balancing, the client gets a list of servers and picks one. Fast and direct, but only practical when you control the clients (or can tolerate slow updates like with DNS).

---

#### Dedicated Load Balancers

**Plain English:** A dedicated load balancer is a server (or hardware appliance) sitting between clients and your backend, doing the routing for them. The client only knows about the load balancer; the existence of N backend servers is hidden.

```
     +--------+      +--------+      +-------------+
     | Client | ---->|   LB   | ---->| backend #1  |
     +--------+      |        |---->-| backend #2  |
                     +--------+ ---->| backend #3  |
                                     +-------------+
```

**Cost:** every request now has one extra network hop. **Benefit:** central control over routing decisions, instant updates when backends come and go, and the load balancer can do clever things like health checks and TLS termination.

Dedicated load balancers come in two flavors based on which OSI layer they understand.

---

##### Layer 4 Load Balancers

**Plain English:** An L4 load balancer routes purely on TCP/UDP info — source IP, destination IP, source port, destination port. It does NOT look inside the packet. To it, your HTTP request is just an opaque chunk of bytes.

**[ANALOGY]** An L4 load balancer is like a security gate guard who only checks license plates and waves cars through to a numbered parking spot — no peek inside the car.

Key characteristics:

- Keeps the **TCP connection** sticky to one backend for its entire lifetime.
- Very fast (no packet inspection beyond headers).
- Can't make decisions based on app data (URL path, cookies, etc.).
- Best when you need raw throughput or persistent connections.

**[REAL-WORLD EXAMPLE]** AWS's NLB (Network Load Balancer) is a Layer 4 LB. Companies that handle WebSocket-heavy workloads (Slack, Discord, Coinbase) use NLB-style L4 balancers because once a WebSocket is established, the connection has to stay glued to one backend.

##### Where to use Layer 4

When you need persistent connections — **WebSockets**, gRPC streams, raw TCP services like databases — or when you need maximum throughput and don't need URL-based routing, reach for Layer 4.

**[INTERVIEW TIP]** If your design uses WebSockets, explicitly say "I'll use a Layer 4 load balancer for the WebSocket fleet." That single sentence shows you know stickiness matters and that an L7 LB might break long-lived connections.

##### Layer 7 Load Balancers

**Plain English:** An L7 load balancer reads the actual application data — for HTTP, that's the method, URL path, headers, cookies, sometimes even the body. It can make routing decisions based on any of that.

**[ANALOGY]** An L7 load balancer is like a hotel concierge who reads your reservation, looks at your preferences, and personally walks you to the right room. Much smarter, but slower.

Key characteristics:

- Terminates the incoming TCP connection at the LB, opens a *new* TCP connection to the backend.
- Can route based on URL (`/api/*` → API servers; `/static/*` → static asset servers).
- Can route based on cookies (e.g., always send user X to the same backend, "sticky sessions").
- Can do compression, TLS termination, request rewriting, authentication.
- More CPU-hungry due to all that inspection.

**[REAL-WORLD EXAMPLE]** AWS ALB (Application Load Balancer), NGINX, Envoy, HAProxy (in L7 mode), and Cloudflare's L7 proxies are all examples. Netflix uses Envoy as an L7 service mesh. Lyft created Envoy in the first place. GitHub uses HAProxy. Most public-facing web traffic at scale flows through an L7 load balancer somewhere.

##### Where to use Layer 7

Almost any HTTP-based service. L7 is the default for REST APIs, websites, microservices APIs, and anything where you want to route based on URL or headers.

**[INTERVIEW TIP]** "I'll have an L7 load balancer (e.g., NGINX or ALB) terminate TLS, route `/api` to the API service and `/static` to the asset service, and forward bearer tokens through." Three sentences, four checkmarks.

**[COMMON MISTAKE]** Putting WebSockets behind a generic L7 load balancer that doesn't support persistent connections. Some L7 LBs *do* support WebSockets explicitly (NGINX, ALB), but you have to enable it. If your interviewer probes here, acknowledging the limitation wins points.

**In Simple Terms:** L4 = fast, dumb routing based on connections. L7 = smarter, slower routing based on the HTTP request itself. Default L7 for HTTP; switch to L4 for WebSockets and other persistent-connection use cases.

---

##### Health Checks and Fault Tolerance

A load balancer's *other* superpower is automatic failover. It keeps pinging each backend with a **health check** — a periodic probe to see if the server's alive and healthy. If a backend stops responding, the LB takes it out of rotation. If it comes back, the LB puts it back in.

Two common health-check styles:

- **TCP health check** — just try to open a TCP connection on a given port. Fast, cheap. Detects crashed servers and pulled network cables.
- **HTTP (L7) health check** — make a real HTTP request like `GET /healthz` and require a `200 OK`. Catches subtler problems — the process is up but the database is unreachable, for instance.

**[REAL-WORLD EXAMPLE]** Every production service at Google, Meta, Amazon, and Netflix exposes a `/healthz` (or `/health`, `/status`) endpoint that performs lightweight internal checks (database ping, cache ping, dependency check) and returns a clean status. The LB calls this every few seconds.

**[INTERVIEW TIP]** Saying "the load balancer runs HTTP health checks every 5 seconds on a `/healthz` endpoint; if 3 consecutive checks fail, the instance is taken out of the pool" sounds like an SRE wrote it. Interviewers love specifics.

**[DEEPER DIVE]** Health checks are a deceptively hard topic. Too aggressive and one slow query pulls a healthy instance out of rotation; too lax and broken instances keep serving traffic for minutes. Real production systems often have *layered* health: a shallow TCP check for "is the process running?" and a deeper L7 check for "is it serving real traffic correctly?".

**In Simple Terms:** A load balancer doesn't just split traffic — it watches for dead backends and routes around them automatically.

---

##### Load Balancing Algorithms

When the load balancer decides where to send a new request, it picks one using an algorithm. The common ones:

- **Round Robin** — server 1, then 2, then 3, then 1 again. Dirt simple; works great when servers are equally powerful and requests are roughly equal in cost.
- **Random** — pick uniformly at random. Effectively the same as round robin in aggregate.
- **Least Connections** — send to whichever server has the fewest active connections. Important for long-lived connections.
- **Least Response Time** — send to whichever server is currently responding fastest. Good when servers have heterogeneous performance.
- **IP Hash** — hash the client's IP and use it to pick a server. Same client → same server every time. Useful for "session persistence" without cookies.

**[REAL-WORLD EXAMPLE]** WhatsApp's WebSocket fleet (where every connected user is one persistent connection) would use **Least Connections** so that one server doesn't accidentally accumulate all the connections during a deploy. A typical stateless web fleet (like the API tier behind Stripe) uses **Round Robin** since requests are short and uniform.

**[INTERVIEW TIP]** A useful rule of thumb: for **stateless** services, round robin is fine. For **persistent-connection** services (WebSockets, SSE), use Least Connections. Drop those two sentences in your design.

##### Real-World Implementations

There are three families of load balancers you'll encounter:

- **Hardware load balancers** — physical appliances like F5 BIG-IP or Citrix NetScaler. Eye-wateringly expensive but capable of *hundreds of millions of requests per second*.
- **Software load balancers** — HAProxy, NGINX, Envoy, Traefik. Run on commodity Linux. Limited per-instance, but you can horizontally scale them too.
- **Cloud load balancers** — AWS ELB/ALB/NLB, GCP Cloud Load Balancing, Azure Load Balancer. Managed, autoscaling, integrated with the rest of the cloud's networking. Almost everyone in the cloud era uses these.

**[INTERVIEW TIP]** Don't spend interview minutes scaling the load balancer itself. If asked, "the cloud provider's managed LB handles hundreds of millions of requests per second" or "we'd use a hardware load balancer like F5 BIG-IP" closes the topic in under 10 seconds.

**In Simple Terms:** Pick round robin by default, Least Connections for persistent connections, and use a managed cloud load balancer — don't reinvent the wheel.

---

## Common Deep Dives and Challenges

We've covered protocols and load balancing. The source's last section is about the production realities that turn into interview "deep-dive" questions.

---

### Regionalization and Latency

#### The physics problem

A signal can't travel faster than light. In fiber-optic cable it actually travels at about ⅔ the speed of light, or ~200,000 km/s. A New York → London round trip (≈5,600 km each way) is about **56 ms** at the speed of light alone — before any switch, router, server, or DNS query gets involved. Add real-world overhead and you're looking at 80–100 ms.

A New York → Sydney round trip? About 200 ms minimum.

```
[NYC client] ──── ocean ──── [Sydney server]
       \                          /
        \    ~200ms RTT minimum  /
         \________ physics ____/
```

No amount of fancy code will beat that. The fix is to put the server closer to the user.

**[ANALOGY]** It's like ordering coffee. If your favorite coffee shop is in Sydney and you live in NYC, you can pay any price, hire any courier — it's still going to take 24 hours. The only fix is to open a branch in NYC.

**[INTERVIEW TIP]** Saying "the speed of light is the real constraint here" out loud, and computing approximate RTTs, is one of the cleanest ways to sound senior. It shows you reason in physics, not just in code.

#### Strategies inside the physics constraints

Two patterns dominate: CDNs and regional partitioning.

---

#### Content Delivery Networks (CDNs)

**Plain English:** A CDN is a global network of caching servers placed in many cities around the world (called **edge locations** or **PoPs** — points of presence). When a user requests something the CDN has cached, the edge server answers from a few miles away instead of from your origin server thousands of miles away.

**[ANALOGY]** Think of CDNs as Amazon's distributed warehouses. Instead of every package shipping from a single Seattle warehouse, Amazon stores common items in warehouses in every major city. When you order, it ships from the nearest one. Network requests work the same way.

```
                          User
                            |
                            v
                +-----------+-----------+
                |  CDN edge in your city|
                |  Has cached? → serve! |
                |  Miss?         |
                |    |           |
                |    v           |
                |  +--------------------+
                |  |  Origin server     |
                |  | (e.g. NYC)         |
                |  +--------------------+
                +-----------------------+
```

**[REAL-WORLD EXAMPLE]** Netflix runs its own CDN, called **Open Connect**. They literally ship physical boxes to internet service providers (Comcast, Verizon, etc.) and place them inside the ISPs' networks. When you watch a show, the video bytes come from inside your ISP — sometimes from a box one block away. That's why 4K Netflix can play instantly while a small website with bad CDN can stutter.

**[REAL-WORLD EXAMPLE]** YouTube uses Google's CDN, which is essentially every Google data center plus thousands of edge nodes. Cloudflare and Fastly are the two big commercial CDN providers everyone else uses. Discord serves all its emoji and avatars through Cloudflare. Vercel/Netlify's deployment is "your static site at the edge by default."

#### Where CDNs help

When your data is:

- **Highly cacheable** (rarely changes per request, or changes slowly).
- **Read much more than written** (e.g., a viral tweet has billions of reads, millions of writes).
- **Requested from many locations** (a global audience).

Images, video, JavaScript bundles, CSS, font files, downloads, hero images — all classic CDN content. Some companies even cache API responses with short TTLs at the edge for personalized-but-not-super-fresh queries.

**[INTERVIEW TIP]** Bring up a CDN early in any design with a global audience and static assets. "Static assets and images go through a CDN like CloudFront / Cloudflare, with the origin in `us-east-1`" is a one-liner that buys you huge user-perceived latency wins.

**[DEEPER DIVE]** CDNs also do **DDoS protection** (absorbing huge attack traffic at the edge before it hits your origin) and **TLS termination** (offloading the SSL handshake from your servers). These are bonus benefits you can mention if the topic deepens.

**In Simple Terms:** A CDN keeps copies of your static content in cities around the world, so users get fast response from a server nearby.

---

#### Regional Partitioning

**Plain English:** Instead of one giant global database/server fleet, split your system into self-contained regional units. Each region has its own data, its own servers, and ideally serves only users in that region. Latency goes down because users and data are co-located; throughput goes up because you've split the load.

**[ANALOGY]** Think of bank branches. Wells Fargo doesn't store every customer's data in one giant vault in San Francisco — each branch keeps records for its local customers. You walk in, the branch knows you, you're out in 5 minutes.

**[REAL-WORLD EXAMPLE]** Uber is the canonical example, and the source calls it out specifically. When you order a ride in Miami, you only care about drivers currently in Miami. You will *never* need a driver in New York. So Uber partitions data by region — each region has its own dispatch service, its own driver database, its own request queue. The Miami region's database is small (a few thousand active drivers) and fast.

```
+---------------------+    +---------------------+
|     US-East region  |    |    US-West region   |
| - drivers DB        |    | - drivers DB        |
| - dispatch service  |    | - dispatch service  |
| - riders DB         |    | - riders DB         |
+---------------------+    +---------------------+
        |                            |
        +------------+---------------+
                     |
              Global services
       (auth, payments, settings, etc.)
```

**[REAL-WORLD EXAMPLE]** DoorDash partitions per metro for delivery dispatch — totally separate problem per city. Spotify hosts users on regional clusters and your home feed is computed in your region. AWS itself is regionally partitioned: an outage in `us-east-1` doesn't take down `eu-west-1`.

**[INTERVIEW TIP]** Whenever the problem is "users in city X need to interact only with city X's data" (ride-hailing, food delivery, local search, regional events), regional partitioning is the right answer. Call it out by name.

**[DEEPER DIVE]** The hard part of regional partitioning is the **cross-region edge cases**: a user who *moves* between regions, a search that should span multiple cities, fraud detection that needs a global view. These are usually solved with a slower "global" layer that asynchronously syncs across regions, on top of the fast regional layer.

**In Simple Terms:** Split your system geographically so each region has its own data and servers — users get fast local responses, and one region's outage doesn't kill the others.

---

### Handling Failures and Fault Modes

The single most important assumption you must *not* make in distributed systems: **the network is not reliable**. Packets get lost. Cables get cut by backhoes. Routers flap. DNS expires mid-request. Servers die.

You design for failure, not against it.

**[INTERVIEW TIP]** "The fallacy of 'the network is reliable' is one of the most dangerous assumptions in distributed systems." If you can articulate this idea once, naturally, in your design — interviewers will mentally promote you.

---

#### Timeouts and Retries with Backoff

The simplest tool: if a request hangs for too long, give up and try again.

**Timeout** — set a hard cap on how long you'll wait. If exceeded, abandon the request and either retry or surface an error. Crucial because without timeouts, one stuck dependency can pile up open connections and crash your whole service ("thread exhaustion").

**Retry** — when something fails transiently (a brief network hiccup, a server being restarted), trying again often succeeds. But naive retries are *dangerous*. If 1,000 clients are all retrying a failing service at full speed, they make the outage worse — they're DDoSing their own dependency.

##### Backoff

**Plain English:** Instead of retrying immediately, wait. And if the next retry also fails, wait *longer*. The most common variant is **exponential backoff**: 1s, 2s, 4s, 8s, 16s, …

This gives the failing service time to recover. But there's still a problem: if *every* client is doing identical exponential backoff and they all failed at the same instant, they'll all retry at the *same* moments — a synchronized stampede.

The fix: **jitter** — a small random offset on every retry. Each client picks a slightly different wait time. The herd disperses.

```
Without jitter (synchronized):
  failed at t=0
  retry at  t=1, t=3, t=7, t=15  (everyone hits at once)

With jitter (smoothed):
  failed at t=0
  retry at  t=1.3, t=3.7, t=8.1, t=15.9  (everyone's slightly offset)
```

**[REAL-WORLD EXAMPLE]** AWS's own SDKs all implement exponential backoff with jitter by default. AWS has a famous blog post on this topic (linked in the source). Stripe's Java client, Google Cloud's libraries, and Kubernetes' control loops all use the same pattern.

**[INTERVIEW TIP]** The magic interview phrase is **"retry with exponential backoff and jitter."** Drop it once when discussing how clients call your service. For senior interviews, you might be asked to elaborate on jitter — say something like "we add ±20% random offset to each delay so all the failing clients don't retry in lockstep."

##### Idempotency

**Plain English:** An operation is **idempotent** if calling it twice has the same effect as calling it once. `GET /users/42` is naturally idempotent — fetching twice doesn't break anything. But what about `POST /payments` for "charge this card $10"? If you naively retry that, you'll charge twice. Ouch.

The fix: **idempotency keys**. The client generates a unique ID for the request (often a UUID), sends it as a header (e.g., `Idempotency-Key: a1b2c3...`), and the server records that key alongside the operation. If the same key arrives again, the server says "I've already done that — here's the original result" instead of re-running the operation.

```
Client                                Server
  |                                    |
  | POST /payments + Idempotency-Key: K|
  | body: { charge $10 }               |
  |---------------------------------> |
  |                                    | sees K is new, charges $10
  |     200 OK, paymentId=xyz          | records (K -> xyz)
  |<-----------------------------------|
  |                                    |
  | (network blip — client retries)    |
  |                                    |
  | POST /payments + Idempotency-Key: K|
  | body: { charge $10 }               |
  |---------------------------------> |
  |                                    | sees K already used,
  |     200 OK, paymentId=xyz          | returns same result
  |<-----------------------------------|
```

**[REAL-WORLD EXAMPLE]** Stripe's API supports idempotency keys on every write endpoint. Their developers explicitly recommend clients always send one. Same with Shopify's checkout API, Square's payments API, and most modern fintech APIs.

**[INTERVIEW TIP]** When you describe any write API in an interview, casually mention "and we'll accept an `Idempotency-Key` header so clients can safely retry on network failure without double-charging." That single line tells the interviewer you've thought about real-world retry semantics.

**[COMMON MISTAKE]** Assuming GETs are *always* idempotent. They usually are at the application level, but counters and analytics can quietly mutate ("this article view count went up"). It's worth thinking through.

**In Simple Terms:** Set timeouts. Retry with exponential backoff and jitter. Make your write APIs idempotent so retries are safe.

---

#### Circuit Breakers

**Plain English:** A circuit breaker is a piece of code that watches a dependency, and if that dependency is failing badly enough, **stops calling it entirely for a while** to give it room to recover. It "trips" like the breaker in your home electrical panel.

**[ANALOGY]** If your kitchen outlet keeps shorting out, your house's circuit breaker pops *off* — cutting power to that outlet. That prevents your wiring from frying. After a minute you can flip it back on. Software circuit breakers work the same way: when a dependency keeps failing, stop calling it for a cooling-off period.

#### The three states

```
       requests succeeding
        ┌───────────────────┐
        │                   │
        v                   │
  +--------+   too many failures    +--------+
  | CLOSED |  --------------------> |  OPEN  |
  +--------+                        +--------+
       ^                                │
       │  success on test request       │ timeout elapses
       │                                v
       │                          +-----------+
       └─────────────────────────-| HALF-OPEN |
        failure on test request   +-----------+
        (back to OPEN)
```

- **Closed** — normal. Requests flow through. The breaker counts failures.
- **Open** — too many failures crossed the threshold. The breaker rejects all requests *immediately* without even trying the dependency. The client gets fast errors back (or a fallback response).
- **Half-Open** — after a cooling-off period, allow *one* test request through. If it succeeds, close the circuit (back to normal). If it fails, snap back to open.

#### Why this matters

Without circuit breakers, a slow or dead dependency creates a **cascading failure**:

1. Service A's DB starts struggling.
2. Service A's requests pile up waiting on the DB, eating threads.
3. Service B (which calls A) starts timing out.
4. Service B's threads pile up too.
5. Eventually the whole system goes down because everyone is stuck waiting on a single broken thing.

This is the **thundering herd** problem the source warns about: as the system tries to recover, all the retries hit at once and the database can't even boot back up.

A circuit breaker stops the cascade by failing fast: instead of waiting 30s for a timeout, A immediately rejects (or returns a degraded response) so its threads stay free.

**[REAL-WORLD EXAMPLE]** Netflix open-sourced **Hystrix**, the original popular circuit breaker library — they built it after a series of cascading outages took down their streaming. Modern alternatives include **resilience4j** (Java), Go's **gobreaker**, and service-mesh proxies like **Istio** and **Linkerd**, which can enforce circuit breakers at the network level without you writing any code.

**[REAL-WORLD EXAMPLE]** When Amazon checkout's recommendation service starts failing, a circuit breaker quickly cuts those calls and the checkout page renders without recommendations. The user still buys the thing. Without that breaker, the checkout page itself would hang for everyone.

**[INTERVIEW TIP]** Bring up circuit breakers when an interviewer probes on reliability, failure modes, or "what if dependency X goes down?" Saying "we'd wrap calls to X in a circuit breaker — fail fast, give X breathing room, and return a cached or degraded response while open" is a high-signal sentence.

##### Where to apply circuit breakers

- Calls to third-party APIs (payments, email, SMS, geocoding).
- Database connections.
- Service-to-service calls in a microservices mesh.
- Any potentially slow operation that could otherwise wedge your service.

**[DEEPER DIVE]** Production circuit breakers are usually paired with **fallbacks** — when the breaker is open, what do you return? Cached data? A default response? An error? Designing the fallback is often where the interesting trade-offs live (stale data vs. clean failure).

**In Simple Terms:** A circuit breaker stops calling a broken dependency once it's clearly failing — failing fast, freeing your threads, and giving the downstream service time to recover. It's how you prevent one failure from snowballing into a system-wide outage.

---

## Wrapping Up

Networking is the substrate beneath every distributed system. You don't have to be a networking specialist to ace a FAANG system design interview, but you do need a working grip on these four things:

1. **The basics** — IP addresses, DNS, the TCP/IP layered model, and what each layer does for you.
2. **Your protocol toolbox** — TCP vs. UDP at Layer 4; HTTP/HTTPS, REST, GraphQL, gRPC, SSE, WebSockets, and WebRTC at Layer 7. Know the trade-offs and *when to reach for each*.
3. **Load balancing** — client-side vs. dedicated; Layer 4 vs. Layer 7; health checks; algorithms.
4. **Production realities** — regionalization, CDNs, timeouts, retries with exponential backoff and jitter, idempotency keys, and circuit breakers.

Every networking decision in your design ripples into latency, throughput, reliability, and security. Interviewers don't usually expect *one* "right" answer — they're listening for whether you can name the trade-offs and pick something defensible. The defaults this lesson recommends (TCP + REST + L7 LB + retries with backoff + idempotency) are perfectly fine for most interview problems. Reach beyond them only when the problem clearly demands it.

**[INTERVIEW TIP]** A clean interview narrative pattern: **state the default, name the alternative, name the trigger that would make you switch.** Example: "I'll start with REST over HTTPS. If we needed sub-millisecond internal latency between services, I'd switch to gRPC. If the client needed real-time bidirectional streaming, WebSockets."

---

### Follow-Up Opportunities

Reading is one way to learn networking. Building is faster.

- **Wireshark** — a free packet sniffer. Install it, load a website, watch the actual DNS, TCP, and HTTP packets fly across your machine. Five minutes with Wireshark teaches more than a chapter of any textbook. (`https://www.wireshark.org`)
- **Network Link Conditioner** — on macOS, installed via Xcode's Additional Tools, lets you simulate slow networks, packet loss, and high latency. Throttle your connection to "Edge cellular, 5% loss" and see how Twitter, your bank's app, and your own side project behave. You'll discover bugs.
- **`curl -v`** — the verbose flag on curl shows the full HTTP request/response, including TLS negotiation. A great way to feel HTTPS from the inside.
- **`dig` and `nslookup`** — manually inspect DNS records. Run `dig google.com` and `dig +trace google.com` to see how DNS resolution actually walks the global tree.
- **`netcat` (`nc`)** — open a raw TCP connection from your terminal, type an HTTP request by hand, and see the response. Wildly clarifying.

Pair this lesson with hands-on tinkering, then move on to the next *Core Concept* in the Hello Interview series (**API Design**), which extends what you learned in the REST and HTTP sections here.

---

## A quick recap table you can scan before an interview

```
Concept        |  Default in interview               |  Reach for ___ when...
---------------+-------------------------------------+--------------------------------
Transport      |  TCP                                |  UDP for live media / telemetry
Web protocol   |  HTTPS                              |  Always HTTPS for public APIs
API style      |  REST + JSON                        |  gRPC inside, GraphQL for flexible
Push to client |  Polling / SSE                      |  WebSockets for bidirectional
Peer media     |  N/A                                |  WebRTC for audio/video calls
LB layer       |  L7                                 |  L4 for WebSockets / TCP services
LB algorithm   |  Round robin                        |  Least Connections for persistent
Reach           |  Region + CDN for static           |  Regional partitioning per region
Retries        |  Exponential backoff + jitter       |  Idempotency keys for writes
Cascading fail |  Timeouts + circuit breakers        |  Hystrix/resilience4j/service mesh
```

Good luck. The mental model you build from this — *layers, protocols, scaling, failure* — will outlast any specific question they throw at you.
