# Aquarium Controller: TODO

Full details (parts checks, wiring, behaviour) are in `PROJECT_CONTEXT.md`.
Prices are from 29 Oct 2026; check price and stock at checkout.

## 🛒 What to buy

### Robocraze (one order, about ₹795 + cheap bits)
- [ ] ESP32 NodeMCU **30-pin** (WROOM-32, CP2102)
- [ ] 1CH **5V** relay board (10A 250VAC, COM/NO/NC)
- [ ] 5V 2A power adapter (barrel plug)
- [ ] PBS-11B 12 mm push button (momentary)
- [ ] 1000µF 25V capacitor (get 2, one spare)
- [ ] Jumper wires: M-F, F-F and M-M
- [ ] Breadboard, full size (830 points)
- [ ] 10k resistors (a few)
- [ ] DC barrel jack → screw-terminal adapter (to match the adapter's plug, usually 5.5×2.1 mm)
- [ ] USB **data** cable for the ESP32 (micro-USB or USB-C, whichever the board has), if you don't have one

### Electrical shop
- [ ] 3-pin plug (6A)
- [ ] 3-core flexible cable, 0.75 mm², 1–2 m
- [ ] 3-pin socket (6A)
- [ ] 2 PVC junction boxes (mains side + electronics)
- [ ] *Optional:* plug-in RCD adapter, if the home has no RCD/ELCB

### Aquarium shop
- [ ] Sponge filter, sized for the tank
- [ ] Small air pump (230V, a few watts)
- [ ] Airline tubing
- [ ] **Check valve** (non-return valve)

### Home / hardware shop
- [ ] 2 identical small screw-cap bottles (one is the feeder, the other gives the second cap)
- [ ] Two-part epoxy + sandpaper
- [ ] 1–2 small screws (bottle bottom → servo arm)
- [ ] Small funnel (or the cut-off top of a plastic bottle)
- [ ] Silica gel sachets
- [ ] Scrap wood/plastic + screws or double-sided foam tape for the bracket

### Already have
- 180° servo, old Dell laptop, 32 GB drive

### Don't buy yet
- USB-to-Ethernet adapter: only if the laptop's Wi-Fi doesn't work with HA OS
- Pen drive: only if the 32 GB drive is a microSD card
- RTC clock module, 3V relay, spare servo: only if needed later

## ✅ What to do

### Step 1: Now, while the parts ship
- [ ] Place the orders (check prices and stock at checkout)
- [ ] Send Claude the missing answers:
  - [ ] Tank size, and freshwater or saltwater?
  - [ ] Feed times, and any days with no feeding?
  - [ ] Laptop model (sticker underneath)
  - [ ] Is the 32 GB drive a pen drive or a microSD card?
  - [ ] Filter runs: is 3 × 15 min at 07:00, 13:00, 19:00 okay?
- [ ] Build the feeder (the servo is already here):
  - [ ] Cap 1: cut an 8×15 mm slot near the **edge**
  - [ ] Cap 2: 2–4 pinholes of 1–1.5 mm near the edge (heated needle)
  - [ ] Sand, then epoxy + screw the bottle's bottom onto the servo **arm**; leave 24 h to cure
- [ ] Check the laptop BIOS (F2 at power-on): is there a **UEFI** boot option?
- [ ] Install Home Assistant on the laptop (Claude guides this step by step)

### Step 2: Parts arrive, bench test (⚠️ NO mains)
- [ ] Install the ESPHome add-on in HA; first upload over USB (Claude provides the config)
- [ ] Wire on the breadboard (Claude sends a wiring diagram):
  - [ ] Servo + capacitor
  - [ ] Button
  - [ ] Relay: VCC to 5V, IN to GPIO 26, 10k to GND, jumper on **H**
- [ ] Test:
  - [ ] Relay clicks on and off **cleanly, every time** (key test for the 5V relay)
  - [ ] Servo goes to rest with the opening up, and does the flip and the shake
  - [ ] Button gives one feed per press
  - [ ] Buttons and settings show up in the HA app
- [ ] Calibrate over paper with each cap: adjust slot size and shake count, start small

### Step 3: Mains wiring (electrician, if unsure)
- [ ] Mains box: plug → relay COM → NO → socket live; neutral and earth straight through; **closed box**, never on the breadboard
- [ ] Test the socket with a **lamp** first, not the filter

### Step 4: Install at the tank
- [ ] Electronics box **beside** the tank, not above it
- [ ] Feeder on the rim, funnel away from the filter
- [ ] Drip loops on all cables
- [ ] Check valve on the air tube; pump plugged into the switched socket
- [ ] Watch the first few days (portions, uneaten food, filter runs) and tweak settings in HA
- [ ] Later: remote access (Tailscale), if wanted
