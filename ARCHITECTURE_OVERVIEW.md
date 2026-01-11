# Bondy Architecture Overview (High-Level)

Bondy is an experimental application-layer transport that aggregates multiple network links while keeping the media pipeline and codecs outside of its scope. This document describes the system at a high level without exposing protocol internals or code details.

## High-level transport model

- TX receives a standard SRT input stream and treats it as an opaque byte stream.
- The stream is segmented and transmitted over multiple uplink interfaces using a custom transport layer.
- The transport layer focuses on reliability and aggregation using ARQ/FEC and adaptive scheduling.
- RX merges packets from multiple paths, applies reordering and recovery, and outputs a standard SRT stream.
- Telemetry is used for monitoring and internal runtime decisions.

For protocol-level specifics, refer to internal documentation only.

## Roles and platforms

- Bondy TX runs on Raspberry Pi 4/5 and ingests a standard SRT input stream.
- Bondy RX runs on Windows and outputs a standard SRT stream for downstream tools.
- Web UI and telemetry are exposed locally for monitoring and configuration.

## Operating modes (high level)

- Single-link mode: all traffic uses one uplink interface.
- Multi-link mode: traffic is distributed across multiple uplink interfaces.
- Adaptive vs fixed: adaptive adjusts usage based on runtime conditions; fixed uses a static policy.

## Reliability principles (high level)

- Application-layer ARQ and FEC are used for loss recovery.
- A reorder buffer is used to handle out-of-order delivery.
- Late data is dropped to preserve bounded latency.

## Link handling (high level)

- Uses available network interfaces detected by the OS.
- Monitors link health and dynamically reacts to degradation or loss.
- Removes unhealthy links from active use and reuses them after recovery.

## Telemetry (categories only)

- Throughput and utilization per link.
- Loss indicators and recovery activity.
- Latency and jitter trends.
- Link status and overall session health.

## Failure behavior (high level)

- If a link fails, traffic shifts to remaining links without stopping the stream.
- Under heavy loss or jitter, the system degrades gracefully to preserve continuity.
- Recovery attempts are opportunistic and bounded by latency goals.

## Deployment and networking summary

- TX uses Ethernet for SRT input and does not use it for uplink traffic.
- TX exposes a local Web UI on port 8088.
- TX uses Wi-Fi as a management access point; uplinks are other physical interfaces.
- RX exposes a local Web UI on port 8018 and outputs a standard SRT listener.
- Default SRT input is UDP port 5001 on TX; RX output port is configurable.

## What Bondy intentionally does NOT implement

Bondy is intentionally scoped as an experimental application-layer transport and does not attempt to solve all problems related to contribution workflows. The following design choices are deliberate:

### No media processing or codec awareness

Bondy does not:

- inspect, decode, or re-encode audio/video,
- parse MPEG-TS, PES, NAL units, or container formats,
- perform bitrate adaptation or transcoding.

All media handling is delegated to upstream and downstream tools. Bondy treats the stream as an opaque byte stream.

### No IP-level or kernel-level bonding

Bondy does not rely on:

- OS-level link aggregation,
- kernel modules,
- MPTCP,
- VPN-based bonding,
- interface-level packet steering.

All multi-link behavior is implemented entirely at the application transport layer, allowing deployment on standard systems without special privileges or kernel changes.

### No link steering guarantees

Bondy does not provide:

- deterministic per-packet path selection guarantees,
- QoS enforcement at the network layer,
- SLA-style bandwidth reservation.

Path usage is adaptive and opportunistic, based on observed runtime behavior rather than predefined policies.

### No encryption beyond underlying transports

Bondy does not introduce an additional encryption layer beyond what is provided by:

- SRT at ingress and egress,
- standard HTTPS for license activation.

The internal transport layer focuses on reliability and aggregation rather than cryptographic protection.

### No automatic failover orchestration

Bondy does not implement:

- external link bring-up / tear-down,
- modem control logic (e.g. AT-command-based management),
- SIM switching or carrier orchestration.

Links are treated as available or unavailable based on system-level network interfaces.

### No production guarantees

Bondy does not aim to provide:

- hard real-time guarantees,
- certified broadcast reliability,
- compliance with broadcast or carrier-grade standards.

The project exists primarily as a learning, experimentation, and prototyping platform.

## High-level transport diagram

Below is a readable ASCII diagram of the transport flow:

```
               +--------------------------+
               |     SRT Source (Encoder) |
               |   (single-link, standard)|
               +-----------+--------------+
                           |
                           |  SRT input
                           |
               +-----------v--------------+
               |         Bondy TX         |
               |  Raspberry Pi 4 / 5      |
               |                          |
               |  - SRT input listener    |
               |  - Stream segmentation   |
               |  - Custom transport      |
               |  - ARQ / FEC             |
               |  - Telemetry             |
               +-----+---------+----------+
                     |         |
        +------------+         +------------+
        |                                   |
+-------v--------+                 +--------v-------+
|   Uplink #1    |                 |   Uplink #2     |
|  LTE / 5G modem|                 |  LTE / Ethernet |
|  High latency  |                 |  Lower latency  |
+-------+--------+                 +--------+-------+
        |                                   |
        +---------------+-------------------+
                        |
        Out-of-order, lossy, heterogeneous paths
                        |
               +--------v-------------------+
               |         Bondy RX           |
               |         Windows            |
               |                            |
               |  - Multi-link receive      |
               |  - Adaptive reorder window |
               |  - ARQ / FEC recovery      |
               |  - Stream reconstruction   |
               +-----------+----------------+
                           |
                           |  SRT output
                           |
               +-----------v--------------+
               |     SRT Receiver         |
               |  (Player / Demux / etc.) |
               +--------------------------+
```
