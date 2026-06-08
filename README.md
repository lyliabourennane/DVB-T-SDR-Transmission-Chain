# Study and Experimental Evaluation of an SDR-Based DVB-T Transmission/Reception Chain

This repository contains the implementation, simulation flowgraphs, and technical documentation for a complete Digital Video Broadcasting - Terrestrial (DVB-T) transmission and reception chain. The system is designed and evaluated using Software Defined Radio (SDR) technology, leveraging open-source GNU Radio Companion tools and National Instruments USRP hardware.

## Project Overview

The core objective of this project is to implement and experimentally evaluate the full physical layer of a DVB-T broadcast chain. The system's performance is investigated across two primary modulation schemes: **16-QAM** and **64-QAM**, both operating in **8K OFDM** mode. 

### Key Features
* **Full Physical Layer Implementation**: Built using GNU Radio and the `gr-dvbt` library.
* **Dual-Mode Evaluation**: Complete simulation and hardware testing of 16-QAM and 64-QAM constellations.
* **Real-World Hardware Validation**: Validated using an NI USRP 2920 platform over an over-the-air wireless channel.
* **MPEG-TS Integrity**: Confirms data integrity by continuously streaming an MPEG Transport Stream (`.ts`) file and successfully recovering it at the receiver.
* **Live Video Streaming Architecture**: Includes an advanced architecture designed to substitute file sources with live UDP streams generated via FFmpeg.

---

## System Architecture & Parameters

The system architecture is strictly symmetrical, mapping every transmitter processing block to its corresponding inverse operation at the receiver chain.

### Technical Configuration

| Parameter | Value | Justification / Notes |
| :--- | :--- | :--- |
| **Modulation** | 16-QAM / 64-QAM | Investigated for capacity vs. robustness tradeoffs |
| **FFT Size** | 8192 (8K mode) | Standard DVB-T broadcast configuration (6817 useful subcarriers) |
| **Guard Interval (CP)** | 1/32 | 256 samples cyclic prefix to mitigate multipath interference |
| **Code Rate** | 2/3 | Punctured convolutional inner code for error protection |
| **Sample Rate / BW** | 4 MHz | Optimized to match host PC processing capabilities |
| **Center Frequency** | 2.48 GHz | Configured based on antenna optimal characteristics |
| **Master Clock Rate** | 100 MHz | Fixed hardware constraint for NI USRP 2920 |

### Processing Blocks Pipeline

1. **Transmitter (TX)**:
   `MPEG-TS Source` ➔ `Energy Dispersal` ➔ `Reed-Solomon Encoder (255,239)` ➔ `Convolutional Interleaver` ➔ `Inner Coder (Rate 2/3)` ➔ `Bit & Symbol Inner Interleavers` ➔ `DVB-T Mapper` ➔ `Reference Signals (Pilot Insertion)` ➔ `OFDM Cyclic Prefixer (IFFT)` ➔ `UHD: USRP Sink`

2. **Receiver (RX)**:
   `UHD: USRP Source` ➔ `OFDM Symbol Acquisition` ➔ `FFT (8192-point)` ➔ `Demod Reference Signals (Channel Equalization)` ➔ `DVB-T Demapper` ➔ `Symbol & Bit Inner Deinterleavers` ➔ `Viterbi Decoder` ➔ `Convolutional Deinterleaver` ➔ `Reed-Solomon Decoder` ➔ `Energy Descrambler` ➔ `MPEG-TS Output File Sink`

---

## Hardware Specifications

Experimental validation was conducted using the **National Instruments USRP 2920** software-defined radio platform with the following operational parameters:
* **Frequency Range**: 50 MHz to 2.2 GHz (Extended up to 2.48 GHz via compatible external antennas)
* **Host Interface**: Gigabit Ethernet
* **TX Port**: TX/RX (Normalized Gain: 1)
* **RX Port**: RX2 (Normalized Gain: 500m)
* **Max TX Output**: +20 dBm
* **Max RX Input**: -15 dBm

---

## Experimental Results & Performance Analysis

### 16-Quadrature Amplitude Modulation (16-QAM)
* **Simulation**: Achieved flawless execution with clean, tightly bound constellation points and 100% video recovery.
* **Hardware**: Highly stable real-time over-the-air transmission. The receiver successfully synchronized, equalized channel distortions using embedded pilot tones, and reconstructed the output video stream (`test_out.ts`) with no degradation.

### 64-Quadrature Amplitude Modulation (64-QAM)
* **Simulation**: Fully validated at the software level, producing a clear 8×8 constellation grid.
* **Hardware Limitations**: Encountered significant host-side throughput restrictions. Because 64-QAM carries 6 bits/symbol (a 50% computational increase over 16-QAM), it triggered severe **underflow (U)** on transmission and **overflow (O)** on reception. The host PC could not maintain real-time processing constraints under Windows scheduling, leading to signal discontinuities and decoding failures.
* **SNR Constraints**: Theoretical analysis shows 64-QAM requires a minimum SNR of $\approx 22$ dB compared to the $\approx 16$ dB required by 16-QAM. Underflow discontinuities pushed the experimental SNR below this decodable threshold.

### Gain Optimization Matrix

* **Low Gain**: Insufficient SNR; the receiver failed to decode the signal.
* **Medium Gain (Optimal)**: Clean, stable constellation grid with successful MPEG-TS parsing.
* **High Gain**: Introduced non-linear distortions and power saturation, resulting in heavy Viterbi/Reed-Solomon decoding errors.

---

## Achievable Throughput Summary

Calculated analytically for a 4 MHz bandwidth system under the project's configurations (8K mode, GI=1/32, Code Rate=2/3):

| Modulation | Bits / Symbol | Spectral Efficiency | Raw Bit Rate | Net Useful Bit Rate |
| :--- | :---: | :---: | :---: | :---: |
| **16-QAM** | 4 bits | $\approx$ 2.22 bit/s/Hz | 12.93 Mbit/s | **8.62 Mbit/s** |
| **64-QAM** | 6 bits | $\approx$ 3.33 bit/s/Hz | 19.39 Mbit/s | **12.93 Mbit/s** |

Both configurations yield a net useful throughput that satisfies the bandwidth demands of standard-definition MPEG video streams (which typically require 3 to 6 Mbit/s).

---

## Authors & Acknowledgments

This project was developed as part of the **SDR-Based Design for Radiocom** course at the **University of Science and Technology Houari Boumediene (USTHB)**, Faculty of Electrical Engineering.

* **Bourennane Lylia Fatma** - *Electrical Engineering Department, USTHB*
