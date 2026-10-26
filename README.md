# ESP32-S3 Local CCTV

Turn an ESP32-S3 camera board into a local Wi-Fi CCTV camera that works with Android CCTV apps such as tinyCam Monitor and IP Cam Viewer.

There's no Wi-Fi password in the code. You set the camera up from your phone over Bluetooth, using the setup page hosted here:

**Setup page:** https://creowaves.github.io/ESP32S3-local-cctv/

## Features

- **Bluetooth setup:** pick your Wi-Fi network and enter the password from your phone. Nothing is hardcoded.
- **Clear errors:** if joining fails, the page says why (wrong password, network not found, no response).
- **Remembers Wi-Fi:** details are saved only after a successful join, and the camera rejoins automatically after power cuts.
- **Live MJPEG stream** for up to 3 viewers at once.
- **Snapshot URL** for apps that use still images.
- **Built-in web page** with live view and camera settings (resolution, quality, brightness, flip, mirror).
- **Status light:** an RGB LED shows what the camera is doing, and can be switched off at night.
- **Face watch:** add people to a watch list. When one of them appears, the camera light flashes yellow and the viewing device records video and saves it to its Downloads.
- **Re-run setup** by holding the BOOT button. Unplugging or pressing RST never resets it.

## Hardware

| Part | Notes |
|---|---|
| ESP32-S3-CAM board (diymore, ESP32-S3-N16R8, CH340, dual USB-C) | Any Freenove-style ESP32-S3 WROOM CAM board with the same pinout works |
| OV2640 or OV3660 camera | Usually supplied with the board |
| WS2812B / NeoPixel RGB LED breakout | For example the SparkFun RGB LED Breakout (WS2812B) |
| USB-C data cable and 5 V USB power supply | Use a cable that carries data, not charge-only |

## Wiring

Only the status LED needs wiring. The camera plugs into the board's ribbon connector.

| LED breakout | ESP32-S3-CAM |
|---|---|
| VCC / 5V | 5V |
| GND | GND |
| DIN | GPIO21 |
| DOUT | not connected |

Keep **GPIO0** free, because it's the BOOT button used to re-run setup.

## Install the firmware

### 1. Arduino IDE

1. Install **Arduino IDE 2.x**.
2. Add the ESP32 boards. Go to **File > Preferences > Additional boards manager URLs** and add:
   `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
3. Open **Tools > Board > Boards Manager**, search **esp32** (by Espressif), and install version **3.x**.
4. Open **Sketch > Include Library > Manage Libraries**, search **NimBLE-Arduino** (by h2zero), and install version **2.x**.

### 2. Board settings (Tools menu)

| Setting | Value |
|---|---|
| Board | ESP32S3 Dev Module |
| USB CDC On Boot | Disabled |
| Flash Size | 16MB (128Mb) |
| Partition Scheme | 16M Flash (3MB APP/9.9MB FATFS) |
| PSRAM | OPI PSRAM |
| Upload Speed | 115200 |
| Port | The USB port labelled **TTL** / UART (shows as `usbserial` or `wchusbserial` on a Mac, `COMx` on Windows) |

### 3. Upload

1. Open `firmware/esp32s3_cctv.ino`.
2. Click **Upload**.
3. Open **Serial Monitor** at **115200 baud** and press **RST**. You should see `Camera ready` and `Bluetooth advertising as CCTV-ESP32`.

## Set up the camera

1. Plug the camera in. The light **breathes blue**.
2. On an Android phone, open the [setup page](https://creowaves.github.io/ESP32S3-local-cctv/) in **Chrome**. On iPhone, use the free **Bluefy** browser, because Safari doesn't support Web Bluetooth.
3. Tap **Connect over Bluetooth** and choose **CCTV-ESP32**.
4. Pick your Wi-Fi network (2.4 GHz only), enter the password, and tap **Join network**.
5. When the light turns **green**, the page shows the camera's stream and snapshot addresses.

Tip: in Chrome's menu, choose **Add to Home screen** to keep the setup page as an app icon.

## Camera addresses

Replace `<ip>` with the IP address shown on the setup page.

| Address | What it is |
|---|---|
| `http://<ip>/` | Live view and camera settings |
| `http://<ip>:81/stream` | MJPEG video stream, used by CCTV apps |
| `http://<ip>/capture` | Single JPEG snapshot |
| `http://<ip>/api/status` | Camera status as JSON |

## Add the camera to a CCTV app

### tinyCam Monitor

1. Tap **+** and choose **Add IP camera**.
2. Set **Camera vendor** to **(Generic URL)**.
3. Set **Request** to **MJPEG**.
4. Enter `http://<ip>:81/stream` as the stream URL.
5. Leave username and password empty, then save.

### IP Cam Viewer

1. Tap **Add camera** and choose **Generic URL**.
2. Enter `http://<ip>:81/stream`.
3. If asked for a snapshot URL, enter `http://<ip>/capture`.
4. Use **Test** to check it, then save.

**Keep the IP address fixed.** In your router's settings, add a **DHCP reservation** (sometimes called "static lease" or "reserved IP") for the camera. Otherwise its address can change after a router restart and the app loses it.

## Status light

| Colour | Meaning |
|---|---|
| Blue, breathing | Waiting for a phone (no Wi-Fi set up) |
| Cyan | Phone connected over Bluetooth |
| Amber, blinking | Joining Wi-Fi |
| Red, fast blinking | Couldn't join Wi-Fi |
| Green | Online, nobody watching |
| Green, breathing | Someone is watching the stream |
| Orange, blinking | Wi-Fi lost, reconnecting |
| White, fast blinking | "Flash the light" was pressed, to find the camera |
| Yellow, flashing | Face watch sees someone on your list |
| Purple, fast blinking | Setup reset with the BOOT button |

To turn the light off at night, untick **Status light on** on the camera's live view page. It still lights up for problems and during setup.

## Face watch

Face watch spots chosen people, flashes the camera light yellow, and records video of them to the device you're watching on.

**Where it runs:**
- **On the camera:** the watch list is stored here, so every phone, tablet or computer sees the same people.
- **On the viewing device:** recognition and recording run in the browser on the device that has the camera page open. The ESP32 alone is too slow for reliable face recognition.

**Add a person**
1. Open the camera page at `http://<ip>/`.
2. Under **People to watch for**, type their name.
3. With them facing the camera in good light, tap **Take face sample** 3 times. Have them turn their head slightly between samples.
4. Tap **Save person**. Up to 10 people can be stored.

**Start watching**
1. Tap **Start watching**. The first time, the device downloads the face recognition files, about 7 MB, so it needs internet access.
2. Known faces get a yellow box with their name. Anyone else shows as "Unknown" and doesn't trigger anything.
3. When someone on the list is seen:
   - the camera light flashes yellow
   - recording starts, with names and a timestamp drawn on the video
   - recording carries on until they've been gone for the time you choose (10 seconds to 1 minute)
4. The video is saved to the device's **Downloads** folder as `cctv_<name>_<date>_<time>.webm` (or `.mp4` on Safari). Long visits are split into 5-minute files.

**Settings** (saved on each viewing device)

| Setting | What it does |
|---|---|
| Match sensitivity | **Strict** gives fewer false matches; **Relaxed** catches more, including at angles or in poor light |
| Keep recording after they leave | How long recording continues after the last sighting |
| Flash the camera light yellow | Turn the yellow alert light on or off |
| Save recordings automatically | If off, recordings stay in the list and you tap **Save** |

**Good to know**
- **Keep the page open.** It must stay open and on screen; phones pause it in the background or with the screen off. For round-the-clock watching, use an old tablet or a computer kept on, with the screen set to stay awake.
- **Best browsers:** Chrome on Android or a computer. Safari can't always record.
- **CCTV apps:** tinyCam Monitor and IP Cam Viewer show the stream but don't run face watch.
- **Accuracy:** recognition works best in good, even light with faces at least about 80 pixels wide. Use 640 × 480 or higher and place the camera at head height.
- **Privacy:** only add people who agree to it. If the camera covers areas beyond your own property, UK data protection rules for domestic CCTV apply.

## Change the Wi-Fi network later

1. Hold the **BOOT** button for **3 seconds** while the camera is running. The light flashes purple, then breathes blue.
2. Open the setup page and connect. You can **Rejoin** the saved network or choose a different one.

Unplugging the camera or pressing **RST** keeps the saved settings.

## Troubleshooting

| Problem | Fix |
|---|---|
| Upload fails with "No serial data received" | Hold **BOOT**, press and release **RST**, release **BOOT**, then upload. Use the **TTL** USB port and a data cable. |
| Upload fails at 921600 baud | Set **Upload Speed** to **115200**. On a Mac, install the WCH CH34x driver. |
| "Sketch too big" | Set Flash Size to **16MB** and Partition Scheme to **16M Flash (3MB APP/9.9MB FATFS)**. |
| Setup page says the browser can't use Bluetooth | Use Chrome on Android, or Bluefy on iPhone. The page must be opened from the https link, not as a downloaded file. |
| CCTV-ESP32 doesn't appear in the list | Check the light breathes blue, and turn phone Bluetooth and Location on. If another phone is connected, disconnect it first. |
| "Network not found" | The ESP32 only sees **2.4 GHz** networks. Move the camera closer to the router. |
| "The router rejected the password" | Retype the password and use **Show** to check it. Passwords are case-sensitive. |
| Face watch says it couldn't download the library | The viewing device needs internet access the first time. Check it isn't on a guest network that blocks downloads. |
| Known person isn't recognised | Add new samples in the light where the camera sits, or set Match sensitivity to Relaxed. |
| Recording doesn't start | Use Chrome. Keep the page in the foreground with the screen on. |
| "Camera storage isn't available" | Set Partition Scheme to **16M Flash (3MB APP/9.9MB FATFS)** and upload again. |
| "Camera not detected" | Unplug the camera, reseat the ribbon cable with the contacts facing the board, and plug it back in. |
| Stream is slow or jerky | Lower the resolution to **640 × 480**, increase the quality number, and move the camera closer to the router. |
| The app shows "Too many viewers" | Up to 3 viewers can watch at once. Close other apps or browser tabs that are watching. |
| The app lost the camera after a router restart | Add a DHCP reservation for the camera in your router. |

## Security

- **Wi-Fi password:** it travels from your phone to the camera over Bluetooth. It's never sent to GitHub or any server.
- **Stream access:** the stream has **no login**, so anyone on your home Wi-Fi can view it. Don't expose ports 80 or 81 to the internet with router port forwarding.
- **Setup page:** it contains no private data, so hosting it publicly is safe.
- **Face watch data:** face fingerprints (numbers describing each face) and a small photo of each person are stored on the camera. They're visible to anyone on your Wi-Fi who opens the camera page. Recordings are saved only on the device that made them.

## Project structure

```
index.html                  Phone setup page (Web Bluetooth)
firmware/esp32s3_cctv.ino   ESP32-S3 firmware
README.md                   This guide
```

## How it works

The firmware runs a Bluetooth Low Energy service with commands to scan for Wi-Fi networks, join a network, rejoin the saved one, forget it, restart, and flash the light. It also reports a status value.

The setup page talks to that service with **Web Bluetooth**. It shows the network list, sends the chosen network and password, and follows the status until the camera is online or reports why it failed.

Once on Wi-Fi, the camera captures frames once and sends each frame to every connected viewer.

Face watch uses the open-source [face-api](https://github.com/vladmandic/face-api) library in the browser:
1. It finds faces and turns each one into a 128-number fingerprint.
2. It compares that fingerprint with the stored people.
3. On a match, it tells the camera to flash its light, using `POST /api/alert`.
4. It records the view with the browser's built-in video recorder.
