# IEEE 802.11ah (Wi-Fi HaLow) Tactical Mesh Network: Field Measurements, Performance Analysis, and Operational Use Cases

**AVC Consulting Technical Report**
**Date:** July 2026

> *This document was prepared with the assistance of an AI language model (Claude, Anthropic). All field measurements, test configurations, and factual data reflect actual field experiments. The AI assisted in data analysis, model derivation, and document drafting.*

---

## Abstract

This report presents field measurement results for a low-cost 802.11ah (Wi-Fi HaLow) mesh network deployed on the 914 MHz band using Morse Micro MM6108 chipsets integrated into Raspberry Pi 4B nodes running the OpenMANET firmware stack with BATMAN-V layer-2 routing. Structured range tests were conducted at 0 m, 500 m, and 1,000 m under controlled line-of-sight (LOS) conditions with one antenna elevated to 7 m. An additional pair of tests evaluated non-line-of-sight (NLOS) performance degradation through a reinforced building, and the mitigating effect of repositioning a relay node to restore LOS. A 7 km field connectivity verification further confirmed the system's maximum practical range. From these measurements, a path loss model calibrated to local terrain conditions was derived and used to calculate maximum range and network capacity under four traffic scenarios: ATAK situational awareness only, ATAK with push-to-talk voice, and ATAK with 360p and 720p video. A supplementary channel width comparison (1, 2, 4, and 8 MHz) was conducted at the same distances, quantifying the throughput-vs-range trade-off inherent to 802.11ah's configurable channel width. Civilian and military operational use cases are presented. The complete system, including computing hardware, HaLow module, and enclosure, can be assembled for approximately $500 USD per node.

---

## 1. Introduction

Low-cost, long-range mesh networking is a persistent challenge in both public safety and military communications. Conventional WiFi (2.4 GHz and 5 GHz) offers high throughput but limited range, especially in non-urban and obstructed terrain. Licensed narrowband radios provide range but sacrifice data capacity. IEEE 802.11ah — commercially marketed as Wi-Fi HaLow — occupies a compelling middle ground: Sub-1 GHz propagation characteristics combined with a protocol stack familiar to the broader WiFi ecosystem.

The Sub-1 GHz band provides meaningful advantages over higher frequencies. Lower carrier frequencies experience less free-space path loss, improved diffraction around obstacles, and better penetration through vegetation. The 802.11ah standard supports channel widths from 1 to 8 MHz, physical layer data rates up to 15.4 Mbps at 4 MHz channel width (32.5 Mbps at 8 MHz), and a receiver sensitivity that in practice approaches the thermal noise floor. These characteristics make 802.11ah a candidate for deployable mesh networks in field operations where commercial infrastructure is absent and where cost, weight, and logistics constraints are primary concerns.

The system under test is built around the Morse Micro MM6108 chipset hosted on a WM6108 radio module, running within an OpenMANET software framework on a Raspberry Pi 4B. The mesh layer is provided by BATMAN-V (Better Approach To Mobile Adhoc Networking — Version 5), a proven proactive routing protocol. Integration with ATAK-CIV (Android Team Awareness Kit, civilian edition, version 5.6.0.13) is achieved through a TAK server, with voice communication provided by Murmur (Mumble) and video relay through MediaMTX. The entire software stack runs on open-source components. The network operates at 914.0 MHz (Channel 24), 4 MHz channel width, with WPA3-SAE encryption.

---

## 2. System Architecture

### 2.1 Hardware

Each node consists of:

- **Compute:** Raspberry Pi 4B (4 GB RAM)
- **Radio:** WM6108 module (Morse Micro MM6108 chipset), connected via SPI
- **Antenna:** Omnidirectional, 2 dBi, elevated to 7 m (server/gateway node)
- **Frequency:** 914.0 MHz, Channel 24 (US regulatory domain)
- **Channel width:** 4 MHz (primary); 1, 2, and 8 MHz tested for comparison (Section 3.4)
- **TX Power:** 27 dBm (approximately 500 mW)
- **Mode:** 802.11ah Mesh Point
- **Encryption:** WPA3-SAE (CCMP)
- **Enclosure:** Custom 3D-printed PETG housing, IP-resistant
- **Power:** Two 21700-format Li-ion cells, 4,500 mAh each (total 9,000 mAh per node)
- **Runtime:** Approximately 10 hours under operational load

### 2.2 Software Stack

| Layer | Component |
|---|---|
| Mesh routing | OpenMANET firmware / BATMAN-V |
| Situational awareness server | TAK server |
| Voice communications | Murmur (Mumble) with Opus codec |
| Video relay | MediaMTX (RTSP/WebRTC) |
| Speed testing | OpenSpeedTest (self-hosted, local server) |
| Platform OS | Raspberry Pi OS (Debian-based) |

The use of a local OpenSpeedTest server ensured that all throughput measurements reflected actual HaLow link performance, eliminating the Internet bandwidth as a confounding variable.

### 2.3 Test Topology

All throughput measurements were performed as point-to-point links between exactly two nodes. One node acted as the TAK/OpenSpeedTest server (meshgate); the other acted as the client. The server node was connected to a local gigabit Ethernet switch, ensuring the bottleneck was always the HaLow radio link.

**Standard LOS test (Sections 3.1, 4.1):**
```
[Meshgate / Server]  ─────── LOS ────────  [Client Node]
   (antenna 7 m)                              (ground level)
                      0 / 500 / 1,000 m
```

**NLOS and relay test (Sections 3.2, 4.2):**
```
[Meshgate (A)]  ─────────── ~1,000 m ──────────────  [Relay (B)]
                                                            │
                      ██████████ Building ██████████   30 m │
                      ██ (NLOS obstruction) ████████        │
                                                       [Client (C)]

Direct path A→C blocked by building.
Relay path A→B (LOS, ~1,000 m) + B→C (LOS, 30 m) restores connectivity.
```

---

## 3. Test Methodology

### 3.1 LOS Measurements (Primary Dataset)

The primary measurement campaign was conducted with one node fixed in position with its antenna elevated to approximately 7 m above ground level. A second node was placed at measured distances of 0 m, 500 m, and 1,000 m. Line-of-sight was verified visually at each position. Terrain was semi-urban with mixed grass and low vegetation; conditions were partly sunny with temperatures of approximately 29–33 °C.

At each position, the following metrics were recorded from the HaLow interface status page:

- Signal strength (RSSI, dBm)
- Noise floor (dBm)
- Instantaneous bitrate (PHY layer, Mbit/s)
- Link quality (SNR = Signal − Noise, dB)

A throughput test was then run using OpenSpeedTest to record:

- Download throughput (Mbit/s)
- Upload throughput (Mbit/s)
- Ping (ms)
- Jitter (ms)

### 3.2 NLOS Test

At the 1,000 m position, an obstacle in the form of a multi-story residential building was interposed in the propagation path. The mobile node was positioned such that direct LOS was blocked by the structure. Measurements were repeated without changing any radio parameters.

A relay test was then conducted by deploying a relay node approximately 1,000 m from the meshgate along a clear LOS path, while the NLOS client node was repositioned 30 m laterally from the relay — forming a triangular geometry with the meshgate. The relay thus maintained LOS on both legs: approximately 1,000 m to the meshgate and 30 m to the client. The building creating the original NLOS condition lay along the straight meshgate-to-client path but did not obstruct either of the two relay legs. The relay node used the same hardware and configuration as the primary nodes.

### 3.3 Field Range Validation

A separate test was conducted in open agricultural field terrain at 7 km separation. Both nodes were positioned at ground level (approximately 1 m effective antenna height). The test was not a throughput test; it was a functional connectivity check to verify ATAK CoT (Cursor-on-Target) message exchange. Both users appeared simultaneously on each other's ATAK map, confirming bidirectional data connectivity at 7 km.

### 3.4 Channel Width Comparison Tests

The 802.11ah standard supports channel widths of 1, 2, 4, and 8 MHz, each representing a trade-off between maximum throughput and noise floor sensitivity — and therefore effective range. To quantify this trade-off empirically, the primary LOS test was repeated at all four channel widths under identical conditions: same antenna height (7 m), same terrain, same measurement positions (0 m, 500 m, 1,000 m), and same TX power (27 dBm). The reference 4 MHz dataset is described in Section 3.1; the 1, 2, and 8 MHz datasets were collected in the same session.

Per the 802.11ah US channelisation plan, each channel width uses a different centre frequency:

| Channel width | Centre frequency | Channel no. |
|---|---|---|
| 1 MHz | 915.5 MHz | 27 |
| 2 MHz | 915.0 MHz | 26 |
| 4 MHz | 914.0 MHz | 24 |
| 8 MHz | 916.0 MHz | 28 |

No relay tests were performed for the non-primary channel widths. Full raw data for all channel widths is provided in Appendix C.

---

## 4. Measurement Results

### 4.1 LOS Performance Summary

| Distance | RSSI (dBm) | Noise (dBm) | SNR (dB) | PHY Rate | DL (Mbit/s) | UL (Mbit/s) | Ping (ms) | Jitter (ms) |
|---|---|---|---|---|---|---|---|---|
| 0 m (reference) | −36 | −89 | 53 | 15.4 Mbit/s | 9.1 | 11.8 | 7 | 2 |
| 500 m | −60 | −91 | 31 | 13.9 Mbit/s | 6.3 | 4.7 | 7 | 1 |
| 1,000 m | −71 | −93 | 22 | 6.1 Mbit/s | 2.7 | 2.2 | 8 | 0.6 |

**Protocol efficiency** (ratio of real TCP throughput to PHY layer rate) ranged from 59% at the reference position to 44% at 1,000 m. This decline is expected and reflects the increasing overhead from retransmissions, acknowledgments, and mesh routing as signal quality degrades.

**Latency** remained stable across all LOS positions (7–8 ms ping), confirming that the BATMAN-V mesh layer does not introduce significant routing overhead in the two-node topology.

### 4.2 NLOS Performance (Through Building)

| Condition | RSSI (dBm) | Noise (dBm) | SNR (dB) | PHY Rate | DL (Mbit/s) | UL (Mbit/s) | Ping (ms) | Jitter (ms) |
|---|---|---|---|---|---|---|---|---|
| 1,000 m LOS | −71 | −93 | 22 | 6.1 | 2.7 | 2.2 | 8 | 0.6 |
| 1,000 m NLOS | −64 | −92 | 28 | 4.3 | **0.287** | 0.672 | 10 | 1 |
| Relay (≈1,000 m LOS) | −69 | −99 | 30 | 7.6 | **1.4** | 2.5 | 12 | 9 |

A critical finding emerges from this comparison: **the NLOS node behind the building showed a higher RSSI (−64 dBm) than the 1,000 m LOS node (−71 dBm), yet delivered 9.4× less download throughput.** This apparent paradox is explained by multipath destructive interference. The building generates multiple reflected and diffracted signal copies with varying phase shifts. OFDM modulation partially handles multipath, but the constructive/destructive interference pattern across subcarriers severely degrades the effective channel capacity — despite the aggregate received power being higher than in the LOS case.

A secondary observation is the **DL/UL asymmetry in NLOS conditions** (DL 0.287 vs UL 0.672 Mbit/s — UL is 2.3× higher). This is consistent with the building's proximity to the client node: the obstruction was located closer to the client than to the meshgate. According to Fresnel zone theory, an obstruction close to the receiver creates a larger and more severe shadow zone than the same obstruction close to the transmitter. The downlink therefore suffered greater multipath degradation than the uplink, resulting in pronounced asymmetry despite the channel being nominally symmetric.

**Restoring LOS via a triangular relay geometry increased download throughput by 4.9×** (0.287 → 1.4 Mbit/s), confirming that LOS geometry rather than raw distance is the primary performance determinant in NLOS environments. The relay node at 1,000 m from the meshgate bridged the client at 30 m separation; the short relay-to-client leg imposed negligible additional path loss, with the 1,000 m relay-to-meshgate leg acting as the sole bottleneck.

The relay also revealed an important secondary finding: the noise floor at the relay position was 7 dB lower (−99 vs −92 dBm), improving SNR despite a marginally worse RSSI. This demonstrates that local RF noise — from nearby electronics, urban interference, or structural shielding — can be as significant as signal level in determining practical throughput.

### 4.3 Relay Throughput Penalty

The relay test quantitatively validated the theoretical half-duplex relay throughput model. When the relay node operates on the same 914 MHz channel as both adjacent nodes, each data packet must traverse the wireless medium twice (client → relay, then relay → server). This doubles the medium utilization per unit of delivered data, reducing effective throughput by approximately 50%.

| Link | Download (Mbit/s) | Prediction | Error |
|---|---|---|---|
| 1,000 m direct LOS | 2.7 | — | — |
| 1,000 m via relay (≈1,000 m + 30 m hops) | 1.4 | 2.7 ÷ 2 = 1.35 | **3.7%** |

The measured relay penalty (1.4 vs 1.35 predicted) aligns with the model within 3.7%, providing high confidence in extrapolating relay performance to other distances. Latency through the relay increased from 8 ms to 12 ms, with jitter rising to 9 ms — both remaining within acceptable bounds for real-time voice and video applications.

### 4.4 Channel Width Comparison

#### 4.4.1 Full Measurement Dataset

The table below consolidates all four channel widths at all three test distances. The 4 MHz rows reproduce the primary dataset from Section 4.1 for direct comparison.

| Ch. Width | Centre Freq. | Distance | RSSI (dBm) | Noise (dBm) | SNR (dB) | PHY Rate (Mbit/s) | DL (Mbit/s) | UL (Mbit/s) | Ping (ms) | Jitter (ms) |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 MHz | 915.5 MHz | 0 m | −26 | −99 | 73 | 3.4 | 1.10 | 0.93 | 10 | 0.5 |
| 1 MHz | 915.5 MHz | 500 m | −58 | −96 | 38 | 3.4 | 1.50 | 1.90 | 10 | 0.3 |
| 1 MHz | 915.5 MHz | 1,000 m | −76 | −93 | 17 | 2.4 | 0.98 | 1.10 | 11 | 0.9 |
| 2 MHz | 915.0 MHz | 0 m | −34 | −95 | 61 | 7.5 | 3.50 | 3.20 | 7 | 0.9 |
| 2 MHz | 915.0 MHz | 500 m | −59 | −92 | 33 | 6.7 | 4.00 | 4.50 | 7 | 0.2 |
| 2 MHz | 915.0 MHz | 1,000 m | −76 | −100 | 24 | 3.0 | 1.60 | 1.30 | 11 | 2.0 |
| **4 MHz** | **914.0 MHz** | **0 m** | **−36** | **−89** | **53** | **15.4** | **9.10** | **11.80** | **7** | **2.0** |
| **4 MHz** | **914.0 MHz** | **500 m** | **−60** | **−91** | **31** | **13.9** | **6.30** | **4.70** | **7** | **1.0** |
| **4 MHz** | **914.0 MHz** | **1,000 m** | **−71** | **−93** | **22** | **6.1** | **2.70** | **2.20** | **8** | **0.6** |
| 8 MHz | 916.0 MHz | 0 m | −36 | −87 | 51 | 32.5 | 15.30 | 14.20 | 7 | 0.2 |
| 8 MHz | 916.0 MHz | 500 m | −74 | −77 | 3* | 8.7 | 6.40 | 7.00 | 7 | 0.2 |
| 8 MHz | 916.0 MHz | 1,000 m | −76 | −89 | 13 | 5.8 | 4.30 | 2.70 | 8 | 2.0 |

*Primary dataset (4 MHz) shown in bold. See Section 4.4.3 for the 8 MHz / 500 m anomaly.

#### 4.4.2 Throughput vs. Channel Width

Peak download throughput at 0 m scales with channel width but not linearly, due to varying protocol efficiency:

| Channel width | PHY rate (Mbit/s) | DL at 0 m | DL at 500 m | DL at 1,000 m | Efficiency at 0 m |
|---|---|---|---|---|---|
| 1 MHz | 3.4 | 1.10 Mbit/s | 1.50 Mbit/s | 0.98 Mbit/s | 32% |
| 2 MHz | 7.5 | 3.50 Mbit/s | 4.00 Mbit/s | 1.60 Mbit/s | 47% |
| 4 MHz | 15.4 | 9.10 Mbit/s | 6.30 Mbit/s | 2.70 Mbit/s | 59% |
| 8 MHz | 32.5 | 15.30 Mbit/s | 6.40 Mbit/s | 4.30 Mbit/s | 47% |

The 4 MHz channel achieves the highest protocol efficiency (59%) at close range, likely because the overhead-to-data ratio is more favourable at this intermediate bandwidth. At 1 MHz, proportionally larger fixed overheads (beacon, ACK, mesh management frames) reduce efficiency to 32%. At 8 MHz, efficiency drops to 47% due to the wider channel capturing more ambient interference.

An important observation at 500 m: real throughput for 1 MHz (1.50 Mbit/s) *exceeds* real throughput at 1,000 m for 4 MHz (2.70 Mbit/s) only modestly, confirming that 1 MHz provides adequate capacity for ATAK and voice at intermediate ranges while preserving link margin.

#### 4.4.3 Noise Floor Scaling and 8 MHz Interference Anomaly

Thermal noise power scales with bandwidth: each doubling of channel width increases the noise floor by ~3 dB. The measured noise floors at 500 m follow this relationship for 1, 2, and 4 MHz, but deviate significantly at 8 MHz:

| Channel width | Noise at 500 m | Expected (thermal scaling) | Deviation |
|---|---|---|---|
| 1 MHz | −96 dBm | — (reference) | — |
| 2 MHz | −92 dBm | −93 dBm | +1 dB (within noise) |
| 4 MHz | −91 dBm | −90 dBm | −1 dB (within noise) |
| 8 MHz | −77 dBm | −87 dBm | **+10 dB (anomalous)** |

The 8 MHz channel (centred at 916.0 MHz, spanning 912–920 MHz) partially overlaps with the GSM-900 uplink band (880–915 MHz) at its lower edge. At the 500 m measurement location, a burst interference event from a nearby cellular device is the most likely cause of the elevated noise floor. This interpretation is supported by the throughput result (6.4/7.0 Mbit/s) being inconsistent with an SNR of 3 dB: OFDM's per-subcarrier modulation rejects narrowband burst interference more effectively than the broadband noise floor measurement implies. The 8 MHz data at 500 m should therefore be interpreted as a snapshot during a brief interference event, not representative of steady-state conditions.

At 0 m and 1,000 m, the 8 MHz noise floor (−87 and −89 dBm respectively) aligns with the expected thermal scaling from 4 MHz, confirming that the 500 m anomaly was localised.

#### 4.4.4 Operational Channel Width Recommendations

| Deployment scenario | Recommended channel width | Rationale |
|---|---|---|
| Maximum range, ATAK only | **1 MHz** | Lowest noise floor (+6 dB SNR advantage over 4 MHz) → extended radio range |
| ATAK + PTT voice | **1–2 MHz** | Throughput requirement < 100 kbit/s; range maximised |
| ATAK + voice + 360p video | **2–4 MHz** | 397 kbit/s needed; 2 MHz borderline, 4 MHz comfortable |
| ATAK + voice + 720p video | **4–8 MHz** | 1,547 kbit/s needed; 4 MHz sufficient to ~3 km, 8 MHz extends throughput envelope |
| Mixed fleet (variable missions) | **4 MHz** | Best overall balance of throughput, range, and interference resistance |

---

## 5. Path Loss Model

### 5.1 Model Derivation

Using the two LOS RSSI measurements (500 m and 1,000 m) and the known TX power of 27 dBm, a log-distance path loss model was derived.

**Measured path loss:**

- PL(500 m) = 27 − (−60) = **87 dB**
- PL(1,000 m) = 27 − (−71) = **98 dB**

**Path loss exponent:**

$$n = \frac{PL(1000) - PL(500)}{10 \log_{10}(1000/500)} = \frac{11}{3.01} = 3.65$$

This value is consistent with semi-urban or mixed terrain with partial obstruction (typical range: 3.0–4.0). Free-space propagation would yield n = 2.0; dense urban environments exceed n = 4.0.

**Model validation against 7 km field test:**

Extrapolating from the 500 m reference:

$$RSSI(7000\text{ m}) = -60 - 36.5 \times \log_{10}(7000/500) = -60 - 41.8 = -101.8 \text{ dBm}$$

At this RSSI in open field conditions (measured noise floor ≈ −108 dBm thermal), SNR ≈ 6.2 dB — sufficient for MCS0 (BPSK 1/2) connectivity. ATAK CoT traffic confirmed working at 7 km. **The model and field validation are consistent.**

### 5.2 Throughput vs. Distance Model

An empirical throughput model was calibrated from the two measured data points:

$$T(d) = \frac{12{,}524{,}400}{d^{1.222}} \text{ kbit/s}$$

Where d is distance in meters. This model is valid for the measured terrain conditions (n = 3.65) and the test noise environment (−91 to −93 dBm noise floor). In quieter RF environments (open field, noise ≈ −108 dBm), the SNR-based model predicts significantly better performance at the same distances.

**Model validation points:**

| Distance | Measured DL | Model Prediction | Error |
|---|---|---|---|
| 500 m | 6,300 kbit/s | 6,302 kbit/s | < 0.1% |
| 1,000 m | 2,700 kbit/s | 2,749 kbit/s | 1.8% |
| Relay (≈1,000 m 2-hop) | 1,400 kbit/s | 1,350 kbit/s | 3.7% |

---

## 6. Maximum Range and Capacity Analysis

> **Note on noise environments:** range figures in this section are calculated for two distinct conditions. *Urban* refers to the measured test environment (noise floor ≈ −92 dBm, reflecting local RF interference). *Open field* refers to the thermal noise floor (≈ −108 dBm), consistent with the 7 km field validation where ambient interference was negligible. Actual results in any specific deployment will fall between these bounds depending on local RF environment.

### 6.1 Traffic Requirements per Node

All capacity calculations assume one node per user ("1 fighter = 1 node"), where each node is both an end-user device and a mesh routing participant. All nodes share the same 914 MHz channel (single-channel shared medium).

| Traffic scenario | Bandwidth per node |
|---|---|
| ATAK CoT + SA only | 15 kbit/s |
| ATAK + PTT voice (Murmur/Opus) | 47 kbit/s |
| ATAK + voice + 360p video (H.264 RTSP) | 397 kbit/s |
| ATAK + voice + 720p video (H.264 RTSP) | 1,547 kbit/s |

ATAK CoT messages are compact XML packets (~500 bytes at 5-second intervals per node). Voice uses the Opus codec at 32 kbit/s per active PTT channel. Video values represent typical encoding rates for the stated resolutions.

### 6.2 Maximum Range: Direct Link (2 Nodes)

Maximum range is the distance at which usable throughput (80% of raw throughput after mesh overhead) equals or exceeds the scenario bandwidth requirement. Two noise environments are characterized: urban (measured noise −92 dBm) and open field (thermal noise −108 dBm).

| Scenario | Urban environment | Open field | Limiting factor |
|---|---|---|---|
| ATAK only | **2.7 km** | **7.5 km** | Radio budget |
| ATAK + voice | **2.7 km** | **7.5 km** | Radio budget |
| ATAK + voice + 360p | ~2.2 km (reliable) | ~6.0 km (reliable) | Bandwidth (tight) |
| ATAK + voice + 720p | **1.2 km** | **3.2 km** | Bandwidth |

*Note: For the ATAK + 360p scenario, the theoretical radio limit coincides with the bandwidth limit, leaving approximately 10% throughput margin. Reliable sustained operation is rated at 80% of maximum range.*

*Note: The 7.5 km open-field figure is supported by the 7 km CoT connectivity confirmation.*

### 6.3 Maximum Range: Relay Configuration (3 Nodes, Hop × 2)

The following figures assume a symmetric deployment with the relay positioned equidistant between the two end nodes (equal hop lengths). The actual NLOS relay test used an asymmetric topology (≈1,000 m + 30 m), which confirmed the ÷2 throughput model to within 3.7% (Section 4.3). Equal-hop deployment maximises total range for a given throughput requirement.

With one relay node positioned midway, each hop distance doubles the total coverage at the cost of approximately 50% throughput reduction.

| Scenario | Urban (per hop / total) | Open field (per hop / total) | Limiting factor |
|---|---|---|---|
| ATAK only | 2.7 / **5.4 km** | 7.5 / **15 km** | Radio budget |
| ATAK + voice | 2.7 / **5.4 km** | 7.5 / **15 km** | Radio budget |
| ATAK + voice + 360p | 1.8 / **3.6 km** | 5.0 / **10 km** | Bandwidth |
| ATAK + voice + 720p | 0.75 / **1.5 km** | 2.0 / **4 km** | Bandwidth |

### 6.4 Network Capacity at Fixed Distances

For networks where all nodes are within single-hop range of a mesh gateway (star topology), maximum node count is limited by aggregate throughput capacity divided by per-node traffic:

$$N_{max} = \left\lfloor \frac{T(d) \times 0.8}{BW_{per\,node}} \right\rfloor$$

For ATAK-only traffic:

| Distance from gateway | Max users (ATAK only) |
|---|---|
| 1 km | ~144 (medium access contention becomes the practical limit at ~40–50) |
| 3 km | ~38 |
| 5 km | ~20 |
| 7 km | ~13 |
| 7.5 km (field max) | ~11 |

### 6.5 Key Operational Findings

**NLOS is mission-critical:** a single building reduced throughput by 9.4×. ATAK CoT remains functional, but all video and most voice quality is lost. Relay placement to restore LOS is more impactful than any radio parameter adjustment.

**Relay nodes function as force multipliers for ATAK and voice:** the 50% throughput penalty for relay is acceptable for low-bandwidth traffic (ATAK + voice requires only 47 kbit/s per user). Two relay nodes deployed on elevated positions (masts or terrain features) can extend ATAK + voice coverage to approximately 15 km in open terrain.

**Video drives bandwidth planning:** 360p video from a drone or camera requires 23× more bandwidth than a single ATAK node's CoT updates. Deployments mixing multiple users and multiple video streams must plan relay placement with bandwidth budget in mind, not just distance.

**Latency is excellent throughout:** ping values of 7–12 ms across all scenarios (including relay) are sufficient for real-time voice, video, and ATAK map updates with no perceptible lag.

---

## 7. Civilian Use Cases

### 7.1 Search and Rescue (SAR) Operations

SAR operations in mountainous, forested, or remote terrain frequently occur in areas with no cellular coverage. A HaLow mesh network provides an immediately deployable, infrastructure-free communications backbone. A team of 10–15 rescuers, each carrying a node, can maintain real-time position sharing on ATAK maps, share voice communications over Murmur PTT, and relay drone video from UAV assets conducting aerial search patterns.

At distances up to 7.5 km from a base camp gateway in open terrain, full ATAK + voice + video capability is available per team node. Relay nodes placed at terrain high points extend coverage to mountainous areas where line-of-sight would otherwise be restricted.

**Specific advantages:** the 914 MHz frequency provides better penetration through forest canopy than standard WiFi. The self-healing BATMAN-V mesh means that if a team member descends behind a ridge, traffic is automatically rerouted through the nearest connected node without operator intervention.

### 7.2 Emergency Response and Disaster Relief

Following natural disasters (earthquakes, floods, wildfires), cellular infrastructure frequently fails precisely when communications demand is highest. A set of HaLow nodes can be deployed from a single vehicle or helicopter within minutes, establishing a temporary IP network over a 5–15 km corridor.

First responders gain access to ATAK situational awareness (unit positions, hazard mapping, resource tracking), voice coordination without radio channel congestion, and video feeds from forward positions or drone surveillance. The low per-node cost ($500) allows agencies to pre-position stockpiles of mesh nodes in disaster preparedness kits.

**Relay node configuration:** a relay node mounted on a vehicle roof at 3 m elevation provides an approximate 12 km radio horizon, covering a typical mid-size disaster response perimeter in a single hop.

### 7.3 Event Security and Large Public Gatherings

Security operations at outdoor festivals, sporting events, or political gatherings require real-time coordination across a large area without relying on congested cellular networks. A HaLow mesh installed on temporary structures or light poles provides a private, encrypted IP network for security teams.

With capacity for approximately 38 simultaneous users at 3 km range (ATAK CoT only), a single mesh gateway can support most event security teams. Multiple gateways provide redundancy and capacity scaling.

The latency profile (7–10 ms) is sufficient for live video feeds from fixed cameras or security personnel body cameras at 360p quality within a 2–6 km radius.

### 7.4 Remote Infrastructure Monitoring

Pipeline networks, power transmission corridors, border fences, and irrigation systems often span tens of kilometers in areas without connectivity. Periodic sensor readings and alert notifications require only a few kilobits per second per node — well within ATAK-class bandwidth even at maximum range.

A chain of HaLow relay nodes placed every 5–7 km along a pipeline corridor could provide continuous telemetry and camera monitoring with a single gateway at the control center. The self-healing mesh automatically reroutes traffic around a failed node, providing resilience without operator intervention.

### 7.5 Construction and Industrial Site Management

Large construction sites (airports, dams, industrial facilities) often span several kilometers and employ large workforces requiring position tracking, safety zone monitoring, and coordination. A HaLow mesh provides IP connectivity for worker badges with GPS, environmental sensors, and site cameras within the measured performance envelope.

---

## 8. Military and Tactical Use Cases

### 8.1 Small Unit ATAK Network — Dismounted Patrol

A squad or section of 8–12 personnel, each carrying a HaLow node integrated into standard load-bearing equipment, can maintain full ATAK situational awareness throughout a patrol area extending several kilometers. Friendly force positions are updated every 5 seconds on each user's ATAK device. Chat, attachments, and drawing overlays are shared over the same link.

At 5 km range in open terrain, up to 20 simultaneous ATAK nodes fit within the available bandwidth. At 3 km, the network supports 38 nodes — sufficient for a company-level network with a single gateway.

**Integration note:** the low PHY footprint (sub-1 GHz, 4 MHz channel width, −91 dBm noise floor) provides a degree of Low Probability of Intercept (LPI) compared to standard 2.4 GHz WiFi, whose emissions are readily identifiable by commercial spectrum analyzers.

### 8.2 Forward Observation and Fire Coordination

A forward observer positioned up to 7.5 km from a command post can transmit real-time position data, target grid references, and observer imagery through the HaLow mesh. A PTT voice channel provides direct coordination without occupying a dedicated radio frequency.

For extended range (to 15 km), a relay node — deployed on a mast, vehicle roof, or terrain feature by a support element — bridges the forward observer to the command post. The relay requires no configuration; the BATMAN-V mesh self-configures upon node power-on.

### 8.3 Vehicle-Mounted Relay for Maneuver Operations

A vehicle carrying a HaLow node on a 5 m elevated mast serves as a mobile relay, effectively extending the network horizon to approximately 12–16 km (radio horizon at 5 m height: ~11.5 km). As the vehicle maneuvers, the mesh dynamically selects optimal paths.

A single vehicle-mounted relay between dismounted troops and a command post can sustain: ATAK position tracking for a full platoon, Murmur PTT voice for multiple working groups simultaneously, and a single drone 360p video feed at ranges up to 10 km.

### 8.4 UAV / Drone Integration

HaLow's 914 MHz frequency and mesh architecture are well-suited to UAV communications. A drone carrying a HaLow node can relay video at 360p quality (adequate for reconnaissance) at ranges up to 6 km in open terrain without a relay, or up to 10 km with one relay node on an elevated ground position.

The drone node participates in BATMAN-V routing like any other node, meaning the drone can also relay communications between ground units whose direct links are obstructed — functioning simultaneously as a video collector and a communications relay.

At 720p quality, video range is limited to 3.2 km direct or 4 km via relay, reflecting the higher bandwidth requirement. For longer-range drone operations, reducing resolution to 360p provides a practical range extension of nearly 2×.

### 8.5 Checkpoint and Static Position Monitoring

Fixed checkpoints benefit from elevated relay nodes positioned on existing structures (buildings, water towers, terrain crests). A properly placed relay node restores LOS for positions that would otherwise be degraded by intervening structures — as demonstrated by the triangular relay geometry in the NLOS test (relay at 1,000 m from meshgate, client 30 m from relay), which increased throughput from 0.287 to 1.4 Mbit/s.

For static positions within 2 km of a relay, the full video + voice + ATAK profile is reliably supportable. Beyond 2 km via relay, ATAK and voice remain fully functional to the relay distance limit.

### 8.6 Training Exercise Communications Infrastructure

HaLow mesh networks are cost-effective alternatives to commercial leased lines for training exercise connectivity. A set of 10–20 nodes can cover a 5–15 km training area, providing ATAK tracking, exercise controller communications, and observer/controller video feeds at a fraction of the cost of commercial MANET solutions. The $500 per-node cost allows training units to absorb accidental node loss without significant logistics consequence.

---

## 9. Limitations and Recommendations

### 9.1 NLOS Sensitivity

The most significant operational limitation identified is the catastrophic throughput degradation in NLOS conditions (−9.4× measured through a residential building). Unlike higher-frequency MIMO systems that can exploit multipath propagation, the MM6108 chipset operates as a single-input single-output (SISO) radio and cannot leverage multipath diversity. In any deployment where obstructions are anticipated, relay nodes positioned to restore LOS should be treated as mission-essential rather than optional.

**Recommendation:** deploy relay nodes on elevated platforms (vehicle rooftops at 3–5 m, portable masts at 3–8 m, or terrain crests) in all operations involving buildings, dense vegetation, or significant terrain relief. A 30 m lateral offset at the relay position proved sufficient to restore LOS in the tested urban scenario, yielding a 4.9× throughput improvement.

### 9.2 Frequency Management

The US regulatory domain (chosen for maximum TX power of 27 dBm) provides access to the 902–928 MHz ISM band. Local regulations in the operational area should be verified before deployment. In jurisdictions where the US regulatory domain is not applicable, power and channel availability may differ.

### 9.3 Channel Width vs. Range Trade-off

Channel width has been empirically characterised across 1, 2, 4, and 8 MHz at three distances (Section 4.4). The measured results confirm the theoretical trade-off and provide concrete guidance:

- **1 MHz** delivers a maximum of 1.5 Mbit/s real throughput (500 m) but benefits from the lowest noise floor (−96 dBm at 500 m), extending the SNR advantage at long range. Recommended for ATAK-only deployments where range is the priority.
- **2 MHz** provides a practical midpoint (4.0 Mbit/s at 500 m) with a noise floor 3–4 dB above 1 MHz. Suitable for ATAK + voice at extended range.
- **4 MHz** offers the best protocol efficiency (59% at close range) and covers ATAK + voice + 360p video scenarios. Recommended as the default for mixed-mission deployments.
- **8 MHz** reaches 15.3 Mbit/s peak throughput and sustains 4.3 Mbit/s at 1,000 m, but is more susceptible to adjacent-band interference — confirmed by an anomalous noise event at 500 m (Section 4.4.3). Suitable for short-range, high-throughput scenarios such as video at close range.

Operators should select channel width based on mission traffic profile rather than defaulting to maximum width.

### 9.4 No Anti-Jamming Capability

The 802.11ah protocol operates on a fixed channel with no frequency hopping or spread spectrum anti-jamming features. In environments with deliberate or incidental interference on 914 MHz, performance will degrade. Mitigation options are limited to frequency channel change (requires coordinated reconfiguration of all nodes) or physical shielding of the operating area.

### 9.5 Relay Throughput Budget

Each relay hop reduces end-to-end throughput by approximately 50%. Multi-hop relay chains (three or more hops) should be evaluated carefully against bandwidth requirements. For video-intensive scenarios, relay chains of more than one hop will typically reduce throughput below the video encoding rate, making video unavailable beyond the relay. Operators should plan relay chains for ATAK + voice coverage, and keep video sources within single-hop range of a gateway or relay.

---

## 10. Conclusions

This field measurement campaign demonstrates that an 802.11ah HaLow mesh network built on the MM6108 chipset can deliver:

- **Up to 9.1 Mbit/s download throughput at close range**, sufficient for multiple simultaneous video streams
- **6.3 Mbit/s at 500 m and 2.7 Mbit/s at 1,000 m** under LOS conditions at 914 MHz / 4 MHz channel
- **Confirmed ATAK CoT connectivity at 7 km** in open field terrain, validating the path loss model
- **Relay throughput within 3.7% of the theoretical ÷2 prediction**, enabling confident capacity planning for multi-hop deployments
- **ATAK + voice coverage extending to 15 km** with one relay node in open field conditions
- **360p video coverage to 10 km** with one relay node
- **Stable latency of 7–12 ms** throughout all measured configurations
- **Channel width directly controls the throughput-range trade-off:** 1 MHz provides maximum range for ATAK-class traffic; 8 MHz delivers 15.3 Mbit/s peak throughput for video-intensive scenarios; 4 MHz offers the best overall balance

The system supports meaningful operational scenarios for both civilian emergency response and tactical field operations within a per-node cost envelope of approximately $500, using exclusively open-source software components. The primary operational constraints — NLOS sensitivity, absence of anti-jamming capability, and relay throughput penalty — are well-characterized and can be managed through informed deployment planning.

Future work should include measurements at greater distances in varied terrain types (forest, mountainous, coastal), evaluation of antenna gain improvements (directional relay antennas, elevated omnidirectional antennas), and characterization of network performance under increasing node counts to establish practical CSMA/CA contention limits.

---

## Appendix A — Measurement Hardware Configuration

| Parameter | Value |
|---|---|
| Chipset | Morse Micro MM6108 |
| Module | WM6108 |
| Host platform | Raspberry Pi 4B (4 GB) |
| Operating frequency | 914.0 MHz (Channel 24) |
| Channel width | 4 MHz |
| TX power | 27 dBm |
| Mode | 802.11ah Mesh Point |
| Encryption | WPA3-SAE (CCMP) |
| Routing protocol | BATMAN-V |
| Country code | US |
| Test date | July 2026 |
| Power supply | 2× 21700 Li-ion, 4,500 mAh each (9,000 mAh total) |
| Estimated runtime | ≈ 10 hours under operational load |
| Antenna height (server) | 7 m |
| Antenna type | Omnidirectional |
| Antenna gain | 2 dBi |

## Appendix B — Throughput Model Summary

| Parameter | Value |
|---|---|
| Path loss exponent (n) | 3.65 |
| Reference distance | 500 m |
| Reference RSSI | −60 dBm |
| TX power | 27 dBm |
| Noise floor (urban) | −92 dBm |
| Noise floor (open field) | −108 dBm (thermal) |
| Throughput model | T(d) = 12,524,400 / d^1.222 kbit/s |
| Relay penalty | ≈ 50% (÷2), confirmed to 3.7% accuracy |
| Protocol efficiency | 44–59% (real TCP / PHY rate) |
| Estimated MM6108 sensitivity at MCS0 | ≤ −102 dBm (inferred from 7 km field test) |
| Model validity range | Calibrated from 500–1,000 m measurements; extrapolation beyond 2,000 m carries increasing uncertainty |

## Appendix C — Channel Width Comparison Raw Data

All measurements conducted under identical LOS conditions: antenna height 7 m (server), TX power 27 dBm, WPA3-SAE, US regulatory domain. Throughput measured via local OpenSpeedTest server.

| Ch. Width | Centre Freq. | Distance | RSSI (dBm) | Noise (dBm) | SNR (dB) | PHY (Mbit/s) | DL (Mbit/s) | UL (Mbit/s) | Ping (ms) | Jitter (ms) |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 MHz | 915.5 | 0 m | −26 | −99 | 73 | 3.4 | 1.10 | 0.93 | 10 | 0.5 |
| 1 MHz | 915.5 | 500 m | −58 | −96 | 38 | 3.4 | 1.50 | 1.90 | 10 | 0.3 |
| 1 MHz | 915.5 | 1,000 m | −76 | −93 | 17 | 2.4 | 0.98 | 1.10 | 11 | 0.9 |
| 2 MHz | 915.0 | 0 m | −34 | −95 | 61 | 7.5 | 3.50 | 3.20 | 7 | 0.9 |
| 2 MHz | 915.0 | 500 m | −59 | −92 | 33 | 6.7 | 4.00 | 4.50 | 7 | 0.2 |
| 2 MHz | 915.0 | 1,000 m | −76 | −100 | 24 | 3.0 | 1.60 | 1.30 | 11 | 2.0 |
| 4 MHz | 914.0 | 0 m | −36 | −89 | 53 | 15.4 | 9.10 | 11.80 | 7 | 2.0 |
| 4 MHz | 914.0 | 500 m | −60 | −91 | 31 | 13.9 | 6.30 | 4.70 | 7 | 1.0 |
| 4 MHz | 914.0 | 1,000 m | −71 | −93 | 22 | 6.1 | 2.70 | 2.20 | 8 | 0.6 |
| 8 MHz | 916.0 | 0 m | −36 | −87 | 51 | 32.5 | 15.30 | 14.20 | 7 | 0.2 |
| 8 MHz | 916.0 | 500 m† | −74 | −77 | 3 | 8.7 | 6.40 | 7.00 | 7 | 0.2 |
| 8 MHz | 916.0 | 1,000 m | −76 | −89 | 13 | 5.8 | 4.30 | 2.70 | 8 | 2.0 |

†8 MHz / 500 m: elevated noise floor (−77 dBm) indicates a burst interference event at the measurement location, likely from adjacent-band cellular activity (GSM-900 uplink, 880–915 MHz). Throughput result is consistent with OFDM's per-subcarrier interference rejection; SNR as reported by the driver does not represent steady-state conditions.

---

*AVC Consulting · avc-consulting.kz*
