```
AQUARIUM AUTOMATION (ESP32 IoT) - FULL PROJECT CONTEXT EXPORT
Exported: 30 Oct 2026
Updated:  6 Nov 2026 (single feeder + food setting; owned servo);
          30 Oct 2026 (software route changed to Home Assistant + ESPHome;
          manual button and relay choice decided)
Purpose: hand this to a new Claude workspace so it can continue without re-asking settled questions.

======================================================================
0. HOW TO USE THIS DOCUMENT (instructions for the next Claude)
======================================================================
- This is a DIY ESP32 aquarium controller. Planning is mostly done. No code
  has been written, nothing has been bought, nothing has been wired.
- Section 3 lists DECIDED items. Do not re-open them unless the user does.
- Section 11 lists every question asked so far with the user's answer
  beside it, plus the ones NOT ANSWERED YET. Ask about those; do not
  assume.
- Prices are snapshots from 29 Oct 2026. Re-check before telling the user
  to order.
- Next deliverable once open items are settled: ESPHome firmware config +
  a final wiring diagram/pin map, and Home Assistant setup steps.

======================================================================
1. ABOUT THE USER / WORKING PREFERENCES
======================================================================
- Lively but accurate communication.
- Never assume or make up information. Ask when something is unclear.
- Don't comment on the user's word choices (including swearing).
- The user is fairly new to this. Plain explanations of "why" land well
  (e.g. they asked "why relay?", "why NC?"). Short, direct messages; they
  often reply with brief questions.
- Located in India (shops at robu.in, Robocraze, Amazon.in). Budget is in
  Indian rupees. Mains voltage not confirmed, but very likely 230V.
- Does NOT have access to 3D printing. Mechanical parts must be buildable
  from household items / hardware-shop parts.
- Has no GitHub CLI installed. A GitHub plugin is configured in Claude but
  was failing to connect (auth header error) at the time of export.
  Claude Code (cloud) can commit and push to the project repo
  sweetysharan29-bit/AutoAquariumMaintainer_cld. The user is happy with
  direct pushes to main (no PRs) for this personal repo.
- Owns an old Dell notebook (Celeron, 4 GB RAM, 32 GB eMMC, integrated
  graphics, built-in microSD reader, NO Ethernet port) that will become
  the Home Assistant server. Also has a 32 GB drive (pen drive or microSD:
  not yet confirmed).

======================================================================
2. GOAL AND v1 SCOPE
======================================================================
A DIY IoT aquarium controller built on an ESP32. v1 scope:
  1. Filter: a few SHORT scheduled runs per day, through a single relay
  2. Automatic scheduled feeding
  3. Manual (on-demand) feeding
Feeding must handle TWO food types: flakes, and a fine powder-like food.
Only ONE food is used at a time (see 3.2).

The fish and tank (what we know):
- Small fish. They eat very little and can go days without food.
- The tank currently has NO filter and is doing fine. Water is changed
  about once a month.
- Freshwater vs saltwater: UNKNOWN. Tank size: UNKNOWN.

Key consequence: overfeeding, not underfeeding, is the real risk. Uneaten
food rots and produces ammonia. Portions should be small, with strict
limits. A missed feed is harmless.

======================================================================
3. DECISIONS MADE (with reasoning)
======================================================================

3.1 Board: ESP32 NodeMCU 30-pin (ESP32-WROOM-32, CP2102 USB chip)
- Classic ESP32: most tutorials, supported by ESPHome/Arduino/Blynk/
  RainMaker, plenty of spare GPIO for future sensors.
- 30-pin chosen over 38-pin because the 38-pin is too wide for a standard
  breadboard.
- Rejected: ESP32-C3 (too few pins for future add-ons), ESP32-S3 (overkill
  unless adding a camera), ESP8266 (old, few usable pins).

3.2 Feeder: ONE "pill-bottle drum" feeder on ONE servo (changed 6 Nov 2026)
- The user does not want two separate feeders ("portals"). One container,
  one servo. The user sets in Home Assistant which food is currently in
  the container; the ESP32 stores both dispense routines (flakes = flip,
  powder = shake) and uses the one matching that setting. Two foods are
  never needed at the same time. Saves a servo, a capacitor and wiring.
- ONE BOTTLE, TWO SWAPPABLE CAPS (the user's design, decided 6 Nov 2026):
  the bottle's bottom is permanently fixed to the servo horn; the
  dispensing opening is in the CAP. Two caps: a flakes cap (slot) and a
  powder cap (pinholes). The user screws on the matching cap and sets
  "Food in container" in HA. (Replaces my earlier idea of two bottles
  sharing one cap.) Details in section 8.
- Mismatch risk (cap/food vs setting): flakes motion with the powder cap
  = tiny feed, harmless. Powder shake with the flakes cap, or powder in
  the bottle with the flakes cap on = more food than intended (overfeed).
  Mitigations: every feed notification names the food; the current
  setting is shown on the HA dashboard; daily limits still apply.
(Original plan, kept for history: two feeders, each on its own servo.)
- Why: no 3D printing available; needs to handle flakes AND powder; fish
  need very little food, so rough powder portions are acceptable.
- How the alternatives were ruled out:
  * Single rotating drum: fine for flakes, poor for powder (dumps it).
  * Auger (screw): great for powder, but flakes jam/shred. Building one
    without a 3D printer is fiddly. A vending-machine spring was
    discussed; the user correctly pointed out its open centre and wide
    pitch would leak powder. A spring glued over a solid rod inside a
    tube (~1 mm clearance) would work but needs trial-and-error sourcing.
    Shelved.
  * Pocket wheel: OK for powder, flakes bridge above it.
  * Compartment carousel (round pill organiser on a stepper): handles
    both foods since each portion is pre-loaded; downside is refilling
    every 7-14 feeds. Kept as the FALLBACK if the powder drum turns out
    too inconsistent.
  * Two hoppers (auger + drum): too much build effort without printing.
- Servo, not stepper: the bottle only needs to move within 180 degrees.
  MUST be a 180-degree (positional) servo, NOT the "360 degree continuous
  rotation" version (that one can't hold a position).
- The user ALREADY OWNS 180-degree servos (model SG90/MG90S and count not
  yet confirmed). Tested by the user: goes to commanded angles, returns
  correctly after being moved by hand while off, resists turning when
  powered. But it can be turned a full 360 degrees by hand, so its end
  stop is likely missing/broken. Normal use is fine; avoid turning it by
  hand into the "blind zone" past the sensor's range, and hold the cap
  while unscrewing the bottle. Suggested check: power on from several
  hand-set positions and confirm it always returns smoothly.

3.3 Filter switching: mechanical relay, filter wired through NO contact
- Relay chosen over SSR: most SSRs are normally-open only, and SSRs leak
  a little current when off (can make small motors hum).
- Originally NC was proposed (filter keeps running if the controller
  dies). The user changed this to NO because the tank currently runs fine
  with no filter. With NO: relay idle / controller dead / no power to the
  relay = filter OFF, which matches today's normal state.
- MODULE CHOSEN (30 Oct 2026): Robocraze "1CH 5v Relay Board High/Low
  Level Trigger with Optocoupler", INR 72. The user picked this over the
  recommended 3V module (INR 159) to save INR 87, knowing the risk.
  Consequences:
  * Relay module VCC goes to the 5V rail (adapter +5V), NOT the ESP32
    3V3 pin. A 5V coil won't pull in reliably at 3.3V.
  * The ESP32 still drives IN with 3.3V logic. That is the risk: some 5V
    modules don't switch reliably from a 3.3V signal. Must be verified on
    the bench (relay clicks on and off cleanly, every time) before any
    mains wiring.
  * Fallback if it's unreliable: buy the 3V/3.3V-trigger module (INR 159),
    or add a small NPN transistor stage. Decide only if the test fails.
  * Must still be a 10A 250VAC module with COM/NO/NC terminals (confirm
    on arrival).
- Must be set to HIGH-level trigger (H/L jumper on H).
  Active-low would turn the filter ON while the ESP32 boots.
- 10k ohm pull-down resistor from the relay control GPIO to GND so the
  relay stays off during boot/reset before firmware drives the pin.
- Relay coil draws current from the 5V adapter the whole time the filter
  is on; that's normal and well within the 2A adapter.
- Filter behaviour (updated 6 Nov 2026): SHORT scheduled runs only,
  plus automatic off during feeding. The user: long runs are something
  they can do manually (e.g. plugging the filter in directly), so the
  controller doesn't need to do them. Run times can be any time of day.
  Details in section 9 (FILTER RUNS).
- The relay is a generic switched socket for WHATEVER filter the user
  ends up buying. Requirements for that filter:
  * Draws well under the relay's 10A rating (typical aquarium filters /
    air pumps are a few watts to ~25 W, i.e. well under 1A at 230V).
    Confirm from its label.
  * Restarts by itself after power-off. Most air pumps and internal
    submersible filters do; some hang-on-back and canister filters lose
    their prime and need manual restarting. Avoid those, given the
    frequent on/off.
  * Air-pump type (sponge filter): fit a CHECK VALVE on the air line
    (or keep the pump above water level), so water can't siphon back
    into the pump every time the relay switches it off.
- Filter type the user is considering (6 Nov): one that "sucks in air
  and recirculates water through a filter". Could be a sponge filter +
  air pump (recommended: gentle flow for small fish, electrics stay out
  of the water, low power) or an internal power filter with an air
  intake (stronger flow, motor in the water). Not bought yet.
- User was told (and accepted) the caveat: water sitting in a filter
  while it's off goes stale and is pushed into the tank on restart.
  Several shorter runs per day are gentler than one long one; rinsing
  the filter at monthly water changes helps. Low risk here because the
  filter is mainly for circulation, not the tank's biological
  filtration. With short runs, the sponge's helpful bacteria won't
  establish much; accepted, since the tank already does fine with no
  filter.

3.4 Logic lives on the ESP32
- Whatever app/cloud is used, all schedules, the filter timer, cooldowns,
  daily limits and failsafes run ON THE ESP32. The app is only a remote
  control. Cloud outage, internet loss or a free-plan change must never
  stop the tank from working. The physical button always works.

3.5 Budget target: about INR 1,250 for electronics
- The user wrote "1.25k"; interpreted as INR 1,250 (not objected to).

3.6 Software/control route: HOME ASSISTANT + ESPHOME (decided 30 Oct 2026)
- ESP32 runs ESPHome firmware (YAML config + small C++ lambdas for the
  feed sequence/limits). All logic still runs ON THE ESP32 (section 3.4);
  the tank keeps working if the HA laptop is off.
- Home Assistant OS installed on the user's old Dell notebook (wiped; the
  user agreed). It sits near the router, lid closed, plugged in 24/7.
- Phone control via the Home Assistant companion app. Push notifications
  via the app (no WhatsApp/Telegram needed).
- ESPHome's built-in web_server can give the ESP32 its own fallback web
  page in case the laptop is down.
- Settings (feed times, food in container, fasting days, filter
  run times + run length, limits) are exposed as HA entities and stored ON THE ESP32
  (restore_value), so no app/platform limits apply.
- History: Blynk was chosen first, then dropped because its free plan
  (~5 datastreams) can't hold the settings. Also considered: ESP RainMaker,
  local web page + Telegram, local web page + WhatsApp (CallMeBot =
  one-way alerts only; Meta WhatsApp Business API = paid, overkill).
- HA data stays on the internal eMMC. The user asked about moving it to a
  microSD/32 GB drive; advised against (worse wear, extra failure point,
  32 GB eMMC is enough). Keep backups off the laptop (phone/PC).
- Network: no Ethernet port. Options: laptop Wi-Fi (depends on HA OS
  having a driver for its Wi-Fi chip; configured via console command) or
  a USB-to-Ethernet adapter (Realtek RTL8153) + short cable to the router
  (recommended, not priced).
- Remote access: NOT DECIDED. Tailscale add-on (free) proposed; the user
  will decide later. Alternatives: Nabu Casa / HA Cloud (paid, ~US$6-7/
  month, from memory), Cloudflare Tunnel (needs a domain). Port
  forwarding not recommended (CGNAT common in India, less secure).

3.7 "Food in container" setting drives ALL feeds (decided 30 Oct, updated 6 Nov 2026)
- The user rejected short/long press. Instead: a "Food in container"
  selector in Home Assistant (flakes or powder). Every feed (button,
  app, schedule) uses the dispense routine for the selected food.
- The selection STAYS until the user changes it manually. Store it in the
  ESP32's flash (ESPHome restore_value) so it survives reboots and works
  with no Wi-Fi/HA.
- Reason given by the user: only one food type is in use at a time.

======================================================================
4. FINAL PARTS LIST (final shopping list, 6 Nov 2026; prices from 29 Oct)
======================================================================
The user is buying from this list. No more big design changes.
Prices could not be re-checked on 6 Nov (shop sites blocked from the
Claude session); the user checks price/stock at checkout.

A. ROBOCRAZE (one order, one shipping charge)
  1. ESP32 NodeMCU 30-pin (WROOM-32, CP2102) ........ x1  INR 425
  2. 1CH 5V relay board, High/Low level trigger,
     optocoupler (confirm 10A 250VAC, COM/NO/NC) ..... x1  INR 72
  3. 5V 2A power adapter (barrel plug, usually
     5.5 x 2.1 mm) ................................... x1  INR 183
  4. PBS-11B 12mm panel push button (momentary) ...... x1  INR 19
  5. 1000uF 25V electrolytic capacitor ............... x1  INR 9 (buy 2 for a spare)
  6. Jumper wires M-F 40pcs + F-F 40pcs .............. x1 each INR 87
     PRICED SUBTOTAL .................................................. INR 795
  Same order, not priced (cheap):
  7. Breadboard, full size (830 points) .............. x1
  8. Jumper wires M-M 40pcs .......................... x1
  9. 10k ohm resistors ............................... a few
 10. DC barrel jack (female) to screw-terminal adapter,
     matching the adapter's plug (usually 5.5 x 2.1 mm) x1
 11. USB data cable for the ESP32 (check the board's port: micro-USB
     or USB-C). Skip if owned; it must be a DATA cable, not
     charge-only.

B. ALREADY OWNED
  - 180-degree servo (see 3.2). Backup if unusable: TowerPro MG90S
    180-degree, INR 125, robu.in ("standard quality").
  - 32 GB drive (HA installer if it is a USB pen drive).
  - Old Dell laptop (Home Assistant server).

C. ELECTRICAL SHOP (mains side; generic, suits any small aquarium filter)
  - 3-pin plug (6A) ................................. x1
  - 3-core flexible cable, 0.75 mm2, ~1-2 m ......... x1
  - 3-pin socket (6A) for the filter ................ x1
  - PVC junction box for the mains side ............. x1
  - PVC junction box / enclosure for the electronics  x1
  - Optional: plug-in RCD adapter, if the home has no RCD/ELCB
  - No inline fuse (the user's decision, section 7)

D. AQUARIUM SHOP (filter; recommended type, see 3.3)
  - Sponge filter sized for the tank (tank size still unknown)
  - Small air pump (a few watts, 230V; confirm watts on the label)
  - Airline tubing (length to reach from the pump to the tank)
  - Check valve (non-return valve) for the airline
  - If a different filter is chosen instead: it must restart by itself
    after power-off and draw well under 10A (section 3.3).

E. FEEDER (home / hardware shop)
  - 2 identical small screw-cap bottles (pill bottle / spice jar):
    one is the feeder, the second supplies the SECOND CAP
  - Two-part epoxy + a piece of sandpaper
  - 1-2 small screws (to fix the bottle bottom to the servo horn)
  - Small funnel or cut-off top of a plastic bottle (food chute)
  - Silica gel sachets
  - Scrap wood/acrylic/plastic + screws or double-sided foam tape for the
    servo bracket
  - Tools: small screwdriver, craft knife or small drill (flakes slot),
    needle heated over a flame (powder holes)

F. HOME ASSISTANT LAPTOP (only if needed)
  - USB-to-Ethernet adapter (Realtek RTL8153) + short LAN cable: only if
    the laptop's Wi-Fi doesn't work with HA OS (send laptop model first)
  - USB pen drive 8 GB+: only if the 32 GB drive is a microSD card

G. NOT NOW (optional later)
  - DS3231 RTC module, INR 234 Robocraze (+ LIR2032 rechargeable cell;
    most DS3231 modules try to charge the cell, so a plain CR2032 is not
    suitable unless the charging resistor is removed)
  - 3V/3.3V-trigger relay, INR 159 Amazon.in: ONLY if the 5V module fails
    the bench test (section 3.3)

COST VARIANTS
  - Chosen (5V relay, owned servo, no RTC): INR 795
  - Chosen + RTC: INR 1,029
  - If the 5V relay fails and the 3V module is bought as well: +INR 159

PRICE NOTES / THINGS TO AVOID
  - Robocraze's 180-degree MG90S (INR 129) was OUT OF STOCK. Their in-stock
    "Towerpro MG90S 360 degree" (INR 230) is continuous rotation: AVOID.
  - Other prices seen: ESP32 38-pin Robocraze INR 422, robu.in INR 449;
    28BYJ-48 + ULN2003 Robocraze INR 154 (no longer needed); DS3231 RPi
    mini version Robocraze INR 209.
  - Robocraze prices came from live store data. Amazon.in and robu.in
    prices came from search snapshots and may be stale.

======================================================================
5. SYSTEM OVERVIEW
======================================================================

  5V 2A adapter ---> ESP32 VIN
                ---> Feeder servo (+1000uF cap)
                ---> Relay module VCC
  ESP32 GPIOs ----> servo signal, relay IN; button into a GPIO

  Wall plug Live ---------- Relay COM -> NO -- filter socket Live
  Wall plug Neutral ------------------------- filter socket Neutral
  Wall plug Earth --------------------------- filter socket Earth
  (Relay NC unused)

  All low-voltage grounds tied together (adapter GND, ESP32 GND, servo
  GND, relay GND).

======================================================================
6. PIN MAP (ESP32 NodeMCU 30-pin)
======================================================================
  GPIO 26 -> Relay IN. HIGH = filter on. 10k pull-down to GND.
             (not a strapping pin)
  GPIO 18 -> Feeder servo signal (orange wire), 50 Hz PWM
  GPIO 19 -> Spare (was the second servo; free for a future add-on)
  GPIO 27 -> Manual feed button to GND (internal pull-up, software
             debounce). Feeds using the "Food in container" setting (3.7).
  GPIO 25 -> Spare button (optional, e.g. "run filter now")
  GPIO 2  -> Onboard status LED (on most of these boards; verify)
  GPIO 21 -> RTC SDA (reserved for later)
  GPIO 22 -> RTC SCL (reserved for later)
  VIN     -> Adapter +5V
  GND     -> Common ground
  (Relay VCC is fed from the adapter's +5V rail, not from an ESP32 pin.)

  Deliberately avoided: GPIO 0, 2 (as input), 5, 12, 15 (boot strapping),
  1/3 (USB serial), 6-11 (flash), 34-39 (input-only, no pull-ups).

======================================================================
7. WIRING DETAILS
======================================================================

LOW-VOLTAGE SIDE
  - Adapter +5V -> ESP32 VIN, feeder servo red, relay VCC
  - Adapter GND -> ESP32 GND, servo brown, relay GND
  - One 1000uF capacitor across the servo's +5V/GND, close to the servo
    connector (prevents brownout resets when the servo starts). Polarised:
    stripe side to GND.
  - GPIO18 -> feeder servo orange
  - GPIO26 -> relay IN, plus 10k from GPIO26 to GND
  - Relay H/L jumper on H (high-level trigger)
  - GPIO27 -> button -> GND
  - Servo NEVER powered from the ESP32's 3V3 pin.
  - When uploading over USB, unplug the 5V adapter unless the board is
    confirmed to protect against both being connected. (With the adapter
    unplugged the relay and servo have no power; that's expected.)

MAINS SIDE
  - Switch only the LIVE wire, through relay COM -> NO. Neutral and earth
    go straight through to the filter socket.
  - NO inline fuse (the user's decision, 30 Oct 2026). Protection comes
    from the home's MCB and the RCD/GFCI outlet. Caveat explained to the
    user: the MCB protects the house wiring, not the small relay board, so
    a partial fault in the filter could overheat the relay board without
    tripping the MCB. Low risk with a small filter. Optional middle ground
    the user can choose later: plug the setup into an extension board that
    has a built-in overload breaker.
  - Mains wiring lives inside a closed box, never on a breadboard. Never
    open it while plugged in.
  - Use a GFCI/RCD-protected outlet. Drip loops on every cable.
  - If the user isn't confident with mains wiring, an electrician should
    do that part. Everything else is low-voltage.
  - Relay suitability gets confirmed once the filter's label (watts/amps,
    voltage) is known.
  - Do NOT wire mains until the 5V relay has passed the 3.3V-signal bench
    test (section 3.3).

======================================================================
8. FEEDER MECHANICS (no 3D printing)
======================================================================
- ONE feeder, ONE bottle, TWO swappable caps (see 3.2). The bottle lies
  horizontally, its axis in line with the servo shaft.
- The bottle's BOTTOM is fixed to the servo HORN (arm), not to the servo
  body. The horn comes off with its one screw, so the bottle can still be
  removed for cleaning or a servo swap.
- Fixing: many pill bottles are PP/PE plastic, which superglue and even
  epoxy grip poorly. Roughen both surfaces, use epoxy, and preferably add
  1-2 small screws through the horn's holes into the bottle bottom.
- The dispensing opening is in the CAP, at the free end. Because the
  bottle spins about its own axis, an opening in the CENTRE of the cap
  would never point down. Openings must be NEAR THE EDGE of the cap's
  face or on its side rim, all on one side. A wider cap (spice-jar lid)
  gives more room.
- Optional: tilt the axis 10-15 degrees with the cap end lower, so food
  slides toward the openings (servo bracket mounted at an angle).
- Refill / change food: hold the BOTTLE (not the cap alone) while
  unscrewing, so the twist doesn't go into the servo (the owned servo has
  no end stop). Unscrew cap, refill, screw on the matching cap, then set
  "Food in container" in HA.
- Servo mounted on a simple bracket (wood/acrylic/plastic scrap) on the
  tank rim or lid, a few cm above the water.
- FLAKES cap: one slot/notch near the edge, start around 8 x 15 mm. At
  rest the slot faces UP. Feed = rotate ~180 degrees (slot down), small wiggle to shake
  loose, rotate back up. Portion set by slot size.
- POWDER cap: 2-4 small holes about 1-1.5 mm (heated needle), near the
  edge, on the same side. At rest holes face UP. Feed = rotate holes down, rock about +/-25 degrees N
  times ("salt shaker"), rotate back up. Portion = number of shakes
  (configurable).
- A funnel/chute under the bottle guides food into one spot, away from
  the filter intake.
- Calibration: run feeds over paper first and tune slot size / shake
  count. Start small.
- Humidity: tight caps; silica gel in the feeder housing, NOT loose in the
  bottle (it could fall into the tank). Keep the lid opening under the
  funnel small to limit rising moisture.
- Electronics box sits beside the tank, not over it.
- Powder portions won't be precise; the user accepted this because the
  fish eat very little and can go days without food.

======================================================================
9. BEHAVIOUR SPEC (proposal; all numbers configurable)
======================================================================

NORMAL STATE
- Filter follows its on/off timer (relay on = filter running).
- Servo at rest, opening facing up.
- LED shows Wi-Fi / time status.

FILTER RUNS (updated 6 Nov 2026: short runs only)
- Up to 4 scheduled runs per day, each with its own start time (any time
  of day) and a shared run length. Each run slot can be enabled/disabled.
  PROPOSED DEFAULTS (not confirmed; changeable in HA): 3 runs of 15 min
  at 07:00, 13:00 and 19:00.
- Hard cap on run length: 30 min (PROPOSED). The run-length setting can't
  go above it, so the controller never does long runs. Long runs are done
  manually by the user (e.g. plugging the filter in directly).
- "Run filter now" button in the app (and optionally the spare button):
  one run of the normal run length, then off. Pressing it again during a
  run restarts the countdown, still capped.
- "Stop filter" button: ends the current run early.
- No valid time (no Wi-Fi/NTP, no RTC) -> scheduled runs are skipped,
  filter stays OFF. "Run filter now" still works (it only needs a
  countdown, not the clock).
- Reboot / power cut during a run -> filter stays OFF (no resume).

FEED SEQUENCE (scheduled or manual)
  1. Check limits (cooldown, daily max). If blocked: do nothing, log it,
     blink LED.
  2. If the filter is running, turn it off. Wait ~3 s for flow to settle.
  3. Dispense using the routine for the "Food in container" setting:
     flakes = rotate-tip-return, powder = N shakes.
  4. Keep the filter off for 10 min.
  5. Log the feed. If a filter run was in progress or became due during
     the hold, it resumes for whatever is left of that run (it does NOT
     get extended).

SCHEDULE
- Feed times (e.g. 09:00). No per-slot food choice: every feed uses the
  "Food in container" setting.
- Fasting days: selectable weekdays with no scheduled feeding.
- No valid time -> scheduled feeds are SKIPPED. Manual feeding still
  works. Safe because the fish can go days without food.

MANUAL FEED (decided, section 3.7)
- Physical button: one press = one feed, using the "Food in container"
  setting (flakes or powder). The setting is saved in flash and stays
  until the user changes it in the app.
- Default before the user ever sets it: NOT YET DECIDED (ask; flakes is
  the obvious proposal).
- App "Feed now" button does the same.
- Counts toward the same cooldown and daily limit.

LIMITS
- Cooldown between feeds: 4 h
- Max feeds per day: 2

FAILSAFES
- NO relay wiring: controller dead/unpowered -> filter off (the tank's
  current normal).
- Boot state: relay off (filter off) and servo to rest with opening up,
  before anything else runs; then the timer takes over.
- 10k pull-down keeps the relay off during boot/reset.
- Watchdog: if firmware hangs, the ESP32 reboots into the safe state.
- Feed limits mean button spam or schedule bugs can't overfeed.
- All logic local to the ESP32; the cloud/app is optional.

OPTIONAL
- Notification on each feed ("Fed flakes at 09:00").
- Feed log with timestamps.
- Calibration mode (dispense without counting toward limits, for testing
  on paper).

======================================================================
10. SOFTWARE / CONTROL APP (DECIDED: Home Assistant + ESPHome)
======================================================================
Chosen (30 Oct 2026): ESPHome firmware on the ESP32 + Home Assistant OS on
the user's old Dell notebook + HA companion app on the phone. Details and
reasoning in section 3.6.

HA entities to expose (no platform limits, unlike Blynk):
  - Button: Feed now
  - Buttons: Run filter now, Stop filter
  - Select: Food in container (flakes/powder), persistent on the ESP32
  - Settings: feed times, fasting weekdays, filter run start times (up
    to 4, each enable/disable) + run length (capped at 30 min),
    cooldown, max feeds/day, flake wiggle, powder shake count; all
    stored on the ESP32 (restore_value)
  - Status: filter state, last feed (time + food), feeds today, blocked
    reason, time-valid flag
  - Calibration mode switch (dispense without counting toward limits)
Notifications (e.g. "Fed flakes at 09:00") via an HA automation using the
companion app's push notifications.

HA hardware notes (from memory, re-check against official docs at install):
  - HA OS minimum about 2 GB RAM, 32 GB storage. The laptop meets this.
  - Install: Ubuntu live USB -> write the HA OS "generic x86-64" image to
    the internal eMMC (Disks > Restore Disk Image) -> boot -> open
    http://homeassistant.local:8123. BIOS: UEFI on, Secure Boot off
    (Dell: F2 = BIOS, F12 = boot menu). UEFI support on this laptop not
    yet confirmed.
  - Remote access: undecided, Tailscale proposed (section 3.6).

Options that were discussed and not chosen:
  a) Arduino + built-in web page: free, no server; home Wi-Fi only.
  b) Arduino + Telegram bot: works anywhere, free notifications; settings
     via commands are clunky.
  c) Local web page + WhatsApp: WhatsApp has no free bot API. CallMeBot
     (free, third-party) = one-way alerts only. Meta WhatsApp Business API
     = paid per message, needs business account. Rejected.
  d) Blynk: chosen first, then dropped. Free plan (blynk.io/pricing,
     checked 29-30 Oct 2026): 5 devices, 100,000 messages/month, 5
     events/notifications, 5 automations; datastreams per template unclear
     (5/10/20 cited). Too few datastreams for in-app settings.
  e) ESP RainMaker: free app with built-in schedule UI that runs on the
     ESP32. Good option, but HA chosen since the user has a spare laptop.
  f) Arduino Cloud: free tier 2 devices, 5 variables/device. Rejected.
  g) Adafruit IO: web dashboard rather than a phone app. Not preferred.

======================================================================
11. QUESTIONS & ANSWERS (user's answers recorded beside each question)
======================================================================
ANSWERED (30 Oct 2026)
  Q: Which software/control route?
  A: First picked Blynk. After learning HA could run on a spare laptop,
     chose Home Assistant + ESPHome ("yes" to locking it in). See 3.6.

  Q: Manual button: short press = flakes, long press = powder?
  A: No. Make it a selectable option in the app that stays as set until
     changed manually, because only one food type will be in use. See 3.7.

  Q (6 Nov): Two feeders/servos, or one?
  A: One container, one servo. Tell it in Home Assistant what food is
     inside; the ESP32 remembers how to dispense each type and picks the
     right routine. No need for both foods at once. See 3.2 / 3.7.

  Q (6 Nov): Why a 180-degree servo, not 360?
  A (explained): 180 = positional, holds an angle; 360 = continuous
     rotation, speed only, drifts. The user already owns 180-degree
     servos that work positionally (details in 3.2).

  Q: Relay module: 3V (INR 159, recommended) or 5V (INR 72)?
  A: 5V module, INR 72. See 3.3.

  Q: Save this context doc in the repo and push to main?
  A: Yes. Direct pushes to main, no PRs.

  Q: Is there an always-on computer for Home Assistant?
  A: Old Dell notebook: Celeron, 4 GB RAM, integrated graphics.

  Q: How much free storage does the laptop have?
  A: 32 GB eMMC.

  Q: OK to wipe the laptop?
  A: Yes.

  Q: Does it have an Ethernet port / can it sit near the router?
  A: No Ethernet port, but it can be kept near the router.

  Q (asked by the user): Can HA's data go on a different drive (the
     laptop's built-in microSD reader / a 32 GB drive)?
  A (advice given): Possible, but not recommended (worse wear, extra
     failure point, eMMC is enough). The user didn't object. HA data
     stays on the eMMC.

  Q: Inline fuse on the filter's live wire?
  A: No fuse. Relying on the home MCB + RCD/GFCI outlet. Caveat given
     (see section 7, mains side).

  Q: Remote access via Tailscale?
  A: The user will decide later.

NOT ANSWERED YET (ask the user; do not assume)
  1. Which filter to buy (sponge / internal / hang-on-back), its
     wattage/amps, and mains voltage (confirms the relay is suitable).
     A (6 Nov): Not bought yet. Considering one that "sucks in air and
        recirculates water through a filter" (sponge + air pump, or
        internal filter with air intake). Relay is designed as a
        generic switched socket for whatever is chosen. Still need: link
        or model, watts/voltage, tank size.
  2. Filter run times (times per day).
     A (6 Nov): Short runs only, any time of day; the user does long runs
        manually. Exact times/length not given: proposed defaults are 3 x
        15 min at 07:00, 13:00, 19:00, capped at 30 min. Confirm or
        change (also changeable later in HA).
  3. Feed times; fasting days.
     A: -
  4. Freshwater or saltwater? Tank size? (affects enclosure sealing and
     placement; salt creep corrodes electronics)
     A: -
  5. RTC now or later (currently deferred).
     A: -
  6. Which parts, if any, the user already owns.
     A: Partly: 180-degree servos (count and model SG90/MG90S not yet
        confirmed). Anything else (breadboard, jumpers, adapter)?
  7. Default "Food in container" before the user first sets it in the
     app (proposed: flakes).
     A: -
  8. Remote access: Tailscale (proposed) or other.
     A: Later (see above).
  9. Laptop model (to check its Wi-Fi chip works with HA OS) and whether
     it supports UEFI boot.
     A: -
 10. Laptop Wi-Fi or a USB-to-Ethernet adapter?
     A: - (laptop can sit near the router, so either works)
 11. Is the user's 32 GB drive a USB pen drive or a microSD card?
     A: -
 12. Single-feeder opening: two interchangeable bottles on one cap
     (proposed), or another idea?
     A (6 Nov): The user's own idea, adopted: one bottle permanently
        fixed to the servo arm, two swappable caps (flakes slot / powder
        holes); put on the matching cap and select it in HA. See 3.2/8.

======================================================================
12. FUTURE IDEAS (out of scope for v1)
======================================================================
Temperature sensor (DS18B20) + heater control, LED lighting schedule with
sunrise/sunset, auto top-off with float switches, leak sensor, TDS/pH
probes (isolated modules), dashboards/graphs, ESP32-CAM, dosing pumps.
The spare GPIOs and reserved I2C pins (21/22) leave room for these.

======================================================================
13. SOURCES USED FOR PRICES / PLATFORM LIMITS
======================================================================
- Robocraze store data (ESP32, relays, servos, stepper, DS3231, power,
  buttons, capacitors, wires): https://robocraze.com
- robu.in TowerPro MG90S 180 (standard quality):
  https://robu.in/product/towerpro-mg90s-mini-digital-servo-motor-180-rotation-standard-quality/
- robu.in ESP32 38-pin:
  https://robu.in/product/esp32-38pin-development-board-wifibluetooth-ultra-low-power-consumption-dual-core/
- Amazon.in 1-channel relay listings:
  https://www.amazon.in/1-channel-relay-module/s?k=1+channel+relay+module
- Blynk pricing: https://blynk.io/pricing
- Blynk forum on free-plan limits:
  https://community.blynk.cc/t/blynk-io-free-limits-confusion-how-many-datastreams-per-template-20-or-5/73829
  https://community.blynk.cc/t/essential-changes-to-free-plan/65289
- Arduino Cloud plans: https://cloud.arduino.cc/plans/
- ESP RainMaker scheduling: https://rainmaker.espressif.com/docs/scheduling
- ESP RainMaker in Arduino: https://developer.espressif.com/blog/esp-rainmaker-now-in-arduino/
- Adafruit IO free vs IO+: https://learn.adafruit.com/welcome-to-adafruit-io/io-plus

======================================================================
14. CURRENT STATUS / NEXT STEPS
======================================================================
- Status: hardware and feature design agreed; software route (Home
  Assistant + ESPHome), button behaviour and relay module decided; parts
  list FINAL (section 4); the user is buying parts (6 Nov 2026); no
  code; HA not installed yet.
- Step-by-step checklist (buy + do): TODO.md in the repo root.
- Next steps:
  1. Settle the NOT ANSWERED YET items in section 11.
  2. User buys parts (re-check prices/stock; 180-degree MG90S only).
  3. Deliver the ESPHome config implementing section 9, with all logic on
     the ESP32, plus step-by-step HA OS install on the Dell laptop.
  4. Deliver a final wiring diagram + pin map matching sections 6-7.
  5. Bench test on a breadboard WITHOUT mains: servo, button, relay
     click/LED. Specifically confirm the 5V relay switches reliably from
     the ESP32's 3.3V signal. Calibrate portions over paper.
  6. Mains wiring in a closed box (or by an electrician), then install.
```
