# 07 — Serialization & Deserialization
---

## 1. The Core Problem

A typical web app has two sides:
- **Client** — usually a JavaScript app (React, Angular, Vue)
- **Server** — can be built in Rust, Go, Python, anything

These languages store data in completely different ways internally. A JavaScript object cannot be directly handed to a Rust server — they don't understand each other's internal formats.

**The challenge:** how do two machines with different languages communicate over a network?

---

## 2. The Solution — A Common Standard

Both the client and server agree on a **shared, neutral format** for transmission.

| Term | Definition |
|---|---|
| **Serialization** | Converting native data (JS object, Rust struct) → common format before sending |
| **Deserialization** | Converting received common format → native data after receiving |

This makes communication **language-agnostic** — any machine can talk to any other machine regardless of their underlying tech stack.

```
Client                    Network                   Server
──────                    ───────                   ──────
JS object                                           Rust struct
    │                                                   ▲
    ▼  serialize                       deserialize  │
  JSON  ──────────── bits/packets ────────────────▶  JSON
                                                        │
                                                        │ deserialize
                                                        ▼
                                                   Rust struct
```

---

## 3. The OSI Model (simplified)

When data travels over the internet, it passes through several layers of transformation. This is called the **OSI (Open Systems Interconnection) model**.

| Layer | Name | What it does |
|---|---|---|
| 7 | **Application** | HTTP, JSON — **your responsibility as a backend engineer** |
| 6 | Presentation | Encryption, encoding (SSL/TLS) |
| 5 | Session | Manages connections between apps |
| 4 | Transport | TCP/UDP — segments and port numbers |
| 3 | Network | IP packets — routing between machines |
| 2 | Data Link | Data frames — local network delivery |
| 1 | Physical | Raw 0s and 1s — electrical/optical signals |

### The mental model to keep

> As a backend engineer, **you only work with Layer 7 (Application)**. You send JSON, the layers below handle everything else automatically. By the time data arrives at your server, the network has already reassembled it back into JSON. You never see packets or bits.

Think of it like sending a courier package — you hand over a box, the courier figures out the trucks and planes. You don't need to know how to fly a plane to send a package.

---

## 4. Serialization Standards

There are two families of serialization formats:

### Text-based (human-readable)

| Format | Common use |
|---|---|
| **JSON** | REST APIs — used ~80% of the time |
| **YAML** | Config files (Docker, GitHub Actions) |
| **XML** | Older enterprise systems |

### Binary (compact, fast, not human-readable)

| Format | Common use |
|---|---|
| **Protobuf** | gRPC, high-performance services |
| **Avro** | Big data pipelines |

> **Focus on JSON** — it's the dominant standard for HTTP REST API communication between clients and servers.

---

## 5. JSON Deep Dive

**JSON = JavaScript Object Notation**

Despite the name, it's not limited to JavaScript. Every language can read and write JSON. It's the universal translator for web communication.

### Structure rules

```json
{
  "name": "Aastha",
  "age": 22,
  "active": true,
  "skills": ["Go", "Python", "FastAPI"],
  "address": {
    "city": "Delhi",
    "country": "India"
  }
}
```

| Rule | Detail |
|---|---|
| Must start with `{` and end with `}` | Every JSON object is wrapped in curly braces |
| Keys must be strings in double quotes | `"name":` ✅ — `name:` ❌ |
| Values can only be 5 types | string, number, boolean, array, or nested object |
| Strings use double quotes | `"Aastha"` ✅ — `'Aastha'` ❌ |
| Booleans are lowercase | `true` / `false` ✅ — `True` / `False` ❌ |

### Valid value types

```json
{
  "string_val":  "hello",
  "number_val":  42,
  "boolean_val": true,
  "array_val":   ["a", "b", "c"],
  "object_val":  { "nested": "json" }
}
```

### Where JSON is used

- HTTP REST API request/response bodies ← main focus
- Application log files
- Configuration files

---

## 6. The Complete End-to-End Flow

Here's what happens when a user submits a form on a website:

```
1. CLIENT gathers user input → JS object in memory

2. SERIALIZE (client)
   JS object → JSON string
   Attached to HTTP request body

3. TRANSMIT
   JSON → Data frames → IP packets → 0s and 1s
   (all handled automatically by the network)

4. DESERIALIZE (server)
   0s and 1s → IP packets → Data frames → JSON
   JSON string → Rust struct (or Python dict, Go struct, etc.)

5. BUSINESS LOGIC (server)
   Process data, query database, run logic

6. SERIALIZE (server)
   Rust struct → JSON string
   Attached to HTTP response

7. TRANSMIT (response travels back)

8. DESERIALIZE (client)
   JSON string → JS object

9. UPDATE UI
   Client renders the result for the user
```

> **Pattern to remember:** Serialize before sending, deserialize after receiving. Both sides do both at different points.

---

## 7. Key Takeaways

- **Serialization/deserialization** is the mechanism that lets different languages talk to each other over a network
- **JSON** is the industry standard for REST APIs — human-readable, language-agnostic, strict syntax
- **OSI model** = 7 layers of network infrastructure. You only own Layer 7. The rest is automatic
- The mental model: *client sends JSON → server receives JSON.* Everything in between is handled for you
- Binary formats (Protobuf, Avro) exist for performance-critical scenarios — not typical REST APIs

---
