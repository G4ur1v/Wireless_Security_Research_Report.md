Technical Report: WPA2 Wireless Protocol Security Analysis & Hardware Integration

---------------------------------------

1. Executive Summary
This report details a security assessment of the WPA2 (Wi-Fi Protected Access protocol.

----------------------------------------

2. The study focuses on the 4-Way Handshake vulnerability and the execution of a Man-in-the-Middle (MitM) attack via an Evil Twin Access Point. 
Uniquely, this research utilized an ESP32 microcontroller as the target Access Point to simulate a portable IoT environment. 
All tests were performed in a controlled lab environment on self-owned hardware.

---------------------------------------

3. Target Specifications (The Lab Setup)
Host Access Point: ESP32 Microcontroller Board (configured as a standalone 2.4GHz WiFi Hotspot).

Network SSID: [Your Network Name] (Internal Test Lab).

Encryption: WPA2-PSK (AES).
Operating System: Kali Linux (Rolling Release).

---------------------------------------

4. Lab Infrastructure & Hardware Details
3.1 Wireless Adapters & Chipsets
To ensure successful packet injection and AP hosting, specific hardware was chosen:
Adapter 1 (Monitor/Injection): USB Wireless Adapter with Ralink RT3070 / Atheros AR9271 chipset (High Gain).
Adapter 2 (AP Mode): Secondary adapter for hosting the rogue portal.
3.2 Microcontroller Integration
An ESP32 was programmed using the Arduino IDE to act as the target WiFi node. This allowed for a minimalist, isolated environment to test deauthentication and handshake triggers without interference from commercial routers.

---------------------------------------

5. Technical Challenges: Monitor Mode Troubleshooting
4.1 Problem: Passive Scanning Failure
Initially, attempts to capture traffic using standard managed mode resulted in "Packets Not Found" and "Interface Busy" errors. The OS was unable to "sniff" management frames because the radio was locked to specific SSID association packets.
4.2 The Fix: Forcing Monitor Mode
The issue was resolved by manually killing conflicting processes and forcing the chipset into Monitor Mode.

Command Sequence:

"sudo airmon-ng check kill"

"sudo airmon-ng start wlan0"

"sudo iwconfig wlan0mon mode monitor"


This enabled the adapter to listen to all raw 802.11 frames on the frequency, bypassing the OS-level network stack restrictions.

---------------------------------------

5. Attack Methodology (Step-by-Step)

Step 1: Reconnaissance
Using airodump-ng, the ESP32-hosted network was identified.

Command: airodump-ng wlan0mon

Step 2: Handshake Capture (The 4-Way Handshake)
A Deauthentication Attack was launched against the ESP32 network to force the connected client to re-authenticate. The 4-way handshake (containing the PMKID) was successfully captured in .cap format.

Command: aireplay-ng --deauth 15 -a [ESP32_BSSID] -c [Client_MAC] wlan0mon

Step 3: Evil Twin Deployment
A rogue Access Point was mirrored to match the ESP32 SSID. Using Fluxion/Airgeddon, a DNS Hijacking portal was deployed to intercept HTTP requests.

Step 4: Credential Harvesting
The ESP32 AP was continuously jammed, forcing the victim to migrate to the Evil Twin. 
The credentials entered by the user were captured and instantly verified against the previously captured handshake.

---------------------------------------

6. Technical Analysis & Observations
Chipset Importance: Not all adapters are equal; the Atheros/Ralink chipsets were crucial for stable packet injection.

Management Frames: The success on the ESP32 platform confirms that even micro-hotspots are vulnerable to deauth attacks due to unencrypted management frames in WPA2.

Software Synergy: Kali Linux’s native drivers for wireless pentesting provided the necessary low-level access to the hardware.

---------------------------------------

7. Defensive Recommendations
WPA3 Transition: Implementation of SAE (Simultaneous Authentication of Equals) to prevent handshake sniffing.
PMF (802.11w): Enabling Protected Management Frames to stop deauthentication-based DoS.

Firmware Security: For ESP32/IoT developers, use WPA2-Enterprise or WPA3 where hardware supports it.

---------------------------------------

8. Conclusion
The research successfully demonstrated that WPA2 vulnerabilities persist across all hardware scales—from high-end routers to small ESP32 microcontrollers. 
The integration of specialized chipsets and Kali Linux is essential for identifying these flaws.

---------------------------------------

Prepared By: [Gaurav Khairnar]
Date: 3 April 2026
License: MIT (Educational Use Only)
