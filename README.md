# Bondy – Experimental Custom Transport for SRT Workflows

> **Hobby / experimental engineering project**  
> Not production-ready. No guarantees.

Bondy is an experimental multi-link transport system built as a learning and engineering exercise.  
Bonding is performed by a custom transport layer, while SRT is used only at the input and output edges of the pipeline.

---

## About this project

It is not a commercial product and is not intended to replace professional bonding solutions. It uses a custom transport layer for bonding, while SRT is used only at the input and output edges.

Most of the implementation was developed with the assistance of **AI-based tools**.  
Architecture, design decisions, testing, and integration were performed manually.

---

## Technical details

Bondy uses a custom, project-specific transport protocol optimized for multi-link aggregation. Protocol details are not publicly available at this time.

---

## Bondy TX (Raspberry Pi based transmitter)

### Preparing the image
1. Write the **Bondy TX** image to a microSD card using **Raspberry Pi Imager**.
2. Insert the card into **Raspberry Pi 4 / 5**.
3. Connect:
   - power supply  
   - Ethernet (required for SRT input)  
   - USB modem(s) or other USB network adapters (for uplink)

⚠️ **Strongly recommended:** use a **USB hub with external power** when connecting multiple modems or USB network devices.

### Network and access
- TX creates its own Wi‑Fi access point:
  - **SSID:** `BondyTX-AP`
  - **Password:** `StrongPass123`
- TX also obtains an IP address via LAN if Ethernet is connected.
- **TX Web UI port:** `8088`

### SRT input (Source → TX)
- The SRT source (encoder, player, etc.) must send the stream to:
  - **TX Ethernet interface**
  - **UDP port 5001** (default profile)

⚠️ The Ethernet interface is used **only for receiving the incoming SRT stream** and is not used for uplink transmission.

### Bonding and uplink
- TX distributes traffic over available uplink interfaces:
  - USB LTE/5G modems  
  - USB Ethernet adapters  
  - other supported USB network devices
- Bonding and uplink selection are handled automatically.

#### USB modem support
- Most common USB modems work **plug’n’play**.
- Modems requiring **AT‑command‑based control** are **not supported yet**.

---

## Bondy RX (Windows)

### Preparation
- Extract the **Bondy RX** archive to any folder on a Windows system.

### Launch
1. Run `BondyRX.WebUI.exe`.
2. Open the Web UI:  
   **Port:** `8018` (typically `http://localhost:8018`; ensure the port is not blocked by a local firewall).

### License and activation
The Windows RX application requires a license key to start.
- A **free 60‑day trial** is provided.
- The license is **bound to the local hardware ID**.
- Activation performs a **one‑time HTTPS request** to:  
  `https://lic.mediadelivery.ru/`

After activation, the application does **not** perform background network communication, telemetry, or periodic license checks.

### Receiving and outputting SRT
- RX connects to TX and receives multi-link traffic.
- RX outputs a standard **SRT listener**.
- Connect your player or demultiplexer to:  
  `srt://<RX_IP>:<SRT Port>`

---

## Security notes

This project is an **experimental, hobby‑level engineering project**. It is not intended for production use and does not provide security guarantees.

### Network behavior
Bondy TX and RX do not include telemetry, tracking, or automatic update mechanisms. All network activity is limited to user‑configured SRT/UDP communication.

### License activation
The Windows RX application performs a **single HTTPS request** to the license server during initial activation. After activation, no background network communication or periodic license checks are performed.

### Trust and verification
No hidden services, backdoors, or remote management features are included. Users are encouraged to verify binaries using the provided hash checksums and test the software in isolated environments.

---

## FAQ – Why activation at all?

That’s a fair question. The RX side is protected mainly to limit uncontrolled redistribution while the project is still experimental and to keep testing and feedback manageable at this stage.

The activation is intentionally simple: a one‑time HTTPS request, no telemetry, no background checks, and a fairly long 60‑day trial.

This may change in the future as the project evolves, but for now it’s just a lightweight safeguard rather than a commercial DRM system. If licensing is a blocker for someone testing, I’m happy to discuss options.

---

## Notes
- Experimental project
- No production guarantees
- Behavior may change between versions

Feedback, testing results, and architectural discussion are welcome.
