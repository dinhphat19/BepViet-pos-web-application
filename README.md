#   Bep Viet - web POS and Thermal Receipt Printing System


A lightweight, zero-dependency Point of Sale (POS) web application built for "Bep Viet" restaurant in Bucharest. Designed to handle real-time table management, bilingual kitchen ticket printing, and daily revenue tracking.


## Key Features
-> **Multi-Table State Management:** Tracks 11 dine-in tables ('B1'–'B11') and 5 takeaway slots ('MV1'–'MV5') with persistent state via `localStorage` and automatic daily reset at midnight.

-> **Direct Thermal Printer Integration (QZ Tray & ESC/POS):** Connects via Websocket to local thermal printers to print formatted kitchen tickets with Vietnamese-to-ASCII accent normalization and bilingual (Vietnamese/English) item mapping.

-> **Dynamic Billing & Tipping Engine:** Supports percentage-based (10%, 12%, 15%) and custom tips, calculating net staff tip payouts across Cash (100%) and Card (70%) checkouts.

-> **Daily Revenue Analytics:** Real-time oder history logging and instant Cash vs Card shift summaries.

## Tech Stack
-> **Frontend:** Vanilla JavaScript (ES6+), HTML5, CSS3 Grid/Flexbox
-> **Hardware Integration:** QZ Tray WebSocket API, Raw ESC/POS Commands, Network Sockets

The most critical engineering challenge in this POS system is bridging the gap between a browser-based web application and physical peripheral hardware (thermal receipt printers) deployed in a live restaurant kitchen.

### 1. Communication Layer (QZ Tray & WebSockets)
-> **Problem:** Modern web browsers operate inside a strict security sandbox and block direct TCP/IP socket connections or local hardware port access (`COM`, USB) for security reasons.

-> **Solution:** Integrated **QZ Tray**—a cross-platform desktop background service that acts as a secure bridge. The web app communicates with QZ Tray via an encrypted **WebSocket connection** (`ws://` / `wss://`), allowing the browser to securely query installed system printers and dispatch raw print jobs directly to local hardware.

### 2. Low-Level ESC/POS Byte Stream Construction
Instead of relying on high-level browser print dialogs (which introduce unwanted headers, footers, unpredictable page scaling, and slow print latency), the application generates a **raw ESC/POS byte array** tailored for standard receipt printers:

-> **Initialization & Alignment:** Sends hardware commands (`ESC @` to initialize, `ESC a 0/1` for left/center alignment) to structure the receipt layout.

-> **Typography & Font Control:** Manages font sizing dynamically using command bytes (`ESC !`) to highlight kitchen headers and table numbers.

-> **Automatic Guillotine Cut:** Appends the final hardware cut command (`GS V 0`) to trigger the physical auto-cutter on the thermal printer the moment a ticket is dispatched.

### 3. Vietnamese-to-ASCII Accent Normalization Pipeline
-> **Problem:** Standard kitchen thermal printers rely on hardware code pages (such as CP437 or WPC1250) embedded in their ROM, which often fail to render Unicode Vietnamese diacritics correctly, resulting in garbled characters (mojibake) on busy kitchen tickets.

-> **Solution:** Built a deterministic string normalization function that strips diacritics and maps special Vietnamese characters to standard ASCII equivalents before printing (`ă, â, đ, ê, ô, ơ, ư` -> `a, d, e, o, u`, etc.). This guarantees 100% legibility and eliminates hardware encoding crashes in a high-speed kitchen environment.

### 4. Bilingual Item Mapping & Fault Tolerance
-> **Dual-Language Formatting:** Automatically pairs the local Vietnamese menu name with its English counterpart (e.g., *Phở bò to / Large Beef Pho*) fetched from a centralized menu schema, ensuring smooth communication between front-of-house staff and international kitchen teams.

-> **Connection Fallback & Error Handling:** Implements robust asynchronous promise chains (`RSVP.js` / native Promises) to detect if the QZ Tray service is offline or if a printer is disconnected, gracefully alerting the user via non-blocking UI prompts rather than freezing the order workflow.

EOF