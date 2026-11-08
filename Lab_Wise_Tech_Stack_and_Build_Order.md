# Lab-wise Tech Stack and Build Order

*Companion to **How the Computing World Actually Works** — November 2026*

## How to use this plan

For each numbered topic, complete the short **Build it yourself** exercise in the chapter, then extend the same project into its corresponding lab in Part IX. The lab is the fuller specification, with edge cases, tests, measurements, and a deliverable. Do not do all the chapter exercises first and all the labs afterward.

The recommended order is **1–14 → Chapter 16 capstone → 15–20**. Labs 1–14 form the main path from signals to a working service and machine model. Labs 15–20 explore the additional everyday technologies in Chapter 15. Lab 14 is conceptually useful for the capstone but does not have to be integrated into its code.

## Common setup

- **Environment:** Linux or Ubuntu in WSL on Windows; VS Code or any editor, Git, and a terminal. Keep the networking exercises on localhost or a network you control.
- **C++ projects:** C++20, GCC or Clang, CMake, CTest, and AddressSanitizer/UndefinedBehaviorSanitizer during development. Prefer the standard library and operating-system calls while learning formats and sockets.
- **Python projects:** Python 3, a project virtual environment, `unittest` or `pytest`, NumPy and Matplotlib when numerical work or plots are required.
- **Inspection tools:** Wireshark for your own traffic, `curl` for HTTP, SQLite's command-line tool for inspecting data, and OpenSSL's command-line tools for TLS observations.
- **Cryptography:** PyNaCl for actual encryption, signatures, and key exchange; `argon2-cffi` for application password verifiers. A small-number Diffie–Hellman calculation in Lab 10 is only a teaching model.
- **Media:** Start with self-created PPM images and synthetic segments. Use FFmpeg only for the optional real-media extension of Lab 9 or the capstone.

## Main path: paired chapter builds and labs

| Order | Chapter exercise: build this first | Full lab: extend it next | Recommended technology and additions |
| ---: | --- | --- | --- |
| **1** | Ch. 1 — A noisy channel | **Lab 1 — Physical-channel simulator** | **Python** (`random`, byte arrays); add **NumPy + Matplotlib** for sampling, noise sweeps, bit-error/goodput plots, and repeatable tests. |
| **2** | Ch. 2 — Frame decoder | **Lab 2 — Hex and the local frame** | **C++20** (`std::vector<std::uint8_t>`, bounded offset reads); add **CTest** fixtures, VLAN/options cases, a switch simulator, and **Wireshark** comparison. |
| **3** | Ch. 3 — Tiny router simulator | **Lab 3 — Route a packet through a tiny internet** | **Python** (`dataclasses`, `heapq`); implement longest-prefix matching and hop limits yourself, then add event queues, congestion, failures, and **Matplotlib** plots. |
| **4** | Ch. 4 — Length-prefixed TCP chat | **Lab 4 — TCP chat under adversarial chunking** | **C++20 POSIX sockets** on Linux/WSL; use a small **Python** test client to split, coalesce, delay, and truncate writes. Add CTest cases for protocol parsing. |
| **5** | Ch. 5 — DNS question/answer decoder | **Lab 5 — A DNS decoder that mistrusts every length** | **C++20 binary parser** with offline byte fixtures and CTest; add compression-pointer bounds/cycle checks, response matching, TTL caching, and optional comparison with `dig`/Wireshark. |
| **6** | Ch. 6 — Tiny HTTP server | **Lab 6 — An HTTP server you can explain byte by byte** | Reuse **C++20 sockets** from Lab 4; implement a documented, limited HTTP/1.1 subset. Use **`curl` + browser DevTools** to inspect headers, static assets, and caching. |
| **7** | Ch. 7 — Page-based key/value store | **Lab 7 — A crash-consistent toy store** | **C++20 binary file I/O** and, on Linux/WSL, explicit `fsync` where the format needs durability. Add crash injection, checksums, compaction, and recovery tests. Compare behavior conceptually with SQLite. |
| **8** | Ch. 8 — Toy image codec | **Lab 8 — A codec and its container** | **C++20** for PPM parsing, encoding, decoding, and the container format; **Python + Matplotlib** for compression ratios, reconstruction error, and timing plots. |
| **9** | Ch. 9 — Adaptive segment player simulator | **Lab 9 — A streaming player controlled by buffer seconds** | **Python** for the client, synthetic segments, and localhost server; **NumPy/Matplotlib** for bandwidth traces and buffer/quality timelines. **FFmpeg** is optional for a clip you created. |
| **10** | Ch. 10 — Key-agreement teaching model | **Lab 10 — Authentication is not encryption** | **Python** for small-number, insecure-by-design demonstrations; **PyNaCl** for actual signing/encryption. Observe real TLS using **OpenSSL, `curl`, and Wireshark**. |
| **11** | Ch. 11 — Defensive login lab | **Lab 11 — A login service that checks every boundary** | **FastAPI + Python `sqlite3` + `argon2-cffi`**; add session cookies, parameterized queries, an ownership/access matrix, and automated denial-path tests. Keep Lab 6's raw C++ HTTP server as a separate learning artifact. |
| **12** | Ch. 12 — Lawful protected-media analogy | **Lab 12 — An encrypted file and a temporary license** | **Python + PyNaCl** for authenticated encryption; **FastAPI + SQLite** for an entitlement and temporary-key endpoint. Reuse the Lab 11 account model. |
| **13** | Ch. 13 — Small service under load | **Lab 13 — A service with a database, cache and load test** | **FastAPI + SQLite**, a Python load generator, and **Matplotlib**. Add an index, cache, two workers, background job, idempotency check, and throughput/p95 latency plots. |
| **14** | Ch. 14 — Tiny CPU emulator | **Lab 14 — A CPU emulator with an assembler** | **C++20 + CMake/CTest** for instruction decoding, assembler, memory, interrupts, and trace tests. A tiny text or PPM display is enough; no GUI framework is needed. |

### Integration checkpoint: Chapter 16 capstone

After Lab 14, build **A traceable mini streaming site**. Use **FastAPI + SQLite** for the manifest, segments, sessions, and entitlement; a small **HTML/JavaScript** page for the player and buffer display; **Wireshark, `curl`, and browser DevTools** to inspect requests; and **Python/Matplotlib** to chart bandwidth and stalls. Reuse Lab 9's selection logic and your own media or synthetic bytes. Lab 6's C++ HTTP server can serve a separate simple demonstration, while the capstone uses the application framework for login and database work.

## Chapter 15 extensions: paired mini builds and labs

| Order | Chapter 15 mini build: do this first | Full lab: extend it next | Recommended technology and additions |
| ---: | --- | --- | --- |
| **15** | Pseudorange solver **and** grid touch detector | **Lab 15 — GPS and a sensor-to-event pipeline** | **Python + NumPy + Matplotlib**; add noisy geometry, touch filtering, plots, and a touch event that controls your Lab 9 player. |
| **16** | Code reader and safe action | **Lab 16 — Decode a QR-like grid, then refuse to trust its URL** | **Python + Pillow + `urllib.parse`**; add image rotation/damage, length and checksum validation, and URL review before opening. Start with your own parity scheme; label the format QR-like. |
| **17** | Cell handover simulator | **Lab 17 — A phone moving among three cells** | **Python + NumPy + Matplotlib**; add fading, hysteresis, backhaul limits, interruptions, and a reproducible moving-phone trace. |
| **18** | Pairing state machine | **Lab 18 — Bluetooth-like discovery and pairing** | **Python state machine + PyNaCl**; add stored bonds, key comparison, a simulated middle process, and event-log tests. |
| **19** | Local search engine | **Lab 19 — Crawl, index and rank a private corpus** | **Python standard library** (`pathlib`, `collections`, `re`); add postings, ranking, updates, and benchmark plots. Use only your own/synthetic corpus. |
| **20** | Recommendation from first principles **and** competing-log simulator | **Lab 20 — Two learned vectors and a shared log** | **Python + NumPy** for gradient descent and held-out evaluation; **PyNaCl + `hashlib`** for signed records and a hash-linked, delayed-node simulation. Keep the two programs separate. |

Chapter 15's **“Three small demystifiers”** previews the GPS, QR-like code, and search exercises. Those are already included in orders **15, 16, and 19**; there is no need to implement them twice.

## What to keep for each project

Keep a short README, the exact input or random seed, source code, tests, one example trace or hex dump, and the measurements requested by the lab. When you extend a chapter build into a lab, retain the first version in Git so you can see which new failure cases changed the design.

## Tool documentation

- [CMake tutorial and CTest](https://cmake.org/cmake/help/latest/guide/tutorial/)
- [Python standard library](https://docs.python.org/3/library/), [NumPy](https://numpy.org/doc/stable/), and [Matplotlib](https://matplotlib.org/stable/)
- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [FastAPI SQL database tutorial](https://fastapi.tiangolo.com/tutorial/sql-databases/) and [SQLite documentation](https://www.sqlite.org/docs.html)
- [PyNaCl documentation](https://pynacl.readthedocs.io/en/latest/) and [argon2-cffi password hashing](https://argon2-cffi.readthedocs.io/en/stable/howto.html)
- [FFmpeg documentation](https://ffmpeg.org/documentation.html)
