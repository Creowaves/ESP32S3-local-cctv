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
- **Face watch:** add people to a watch list. When one of them appears, the camera light flashes yellow and video is recorded to the viewing device, the camera's microSD card, or both.
- **microSD card recording:** event videos (AVI) and photos on the camera's card. You can download and delete them from the camera page, and the oldest files are deleted automatically when the card fills up.
- **Re-run setup** by holding the BOOT button. Unplugging or pressing RST never resets it.

## Hardware

| Part | Notes |
|---|---|
| ESP32-S3-CAM board (diymore, ESP32-S3-N16R8, CH340, dual USB-C) | Any Freenove-style ESP32-S3 WROOM CAM board with the same pinout works |
| OV2640 or OV3660 camera | Usually supplied with the board |
| WS2812B / NeoPixel RGB LED breakout | For example the SparkFun RGB LED Breakout (WS2812B) |
| microSD card (optional) | Any size up to 64 GB or more, **formatted as FAT32**; a Class 10 / U1 card or faster |
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

1. Download the `firmware/esp32s3_cctv` folder. Keep `esp32s3_cctv.ino` and `web_page.h` together in a folder named `esp32s3_cctv`, then open the `.ino` file.
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
| `http://<ip>/api/sd/files` | List of files on the memory card |

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

## The camera page

The camera page at `http://<ip>/` has two tabs:

| Tab | What's on it |
|---|---|
| **Camera** | Live view, face watch, recordings on this device, the memory card, picture and speed settings, and the addresses for CCTV apps |
| **Faces** | Live view, adding people (3 face samples each), and the list of people to watch for |

You can open a tab directly with `http://<ip>/#camera` or `http://<ip>/#faces`.

### Making the stream smoother

The **Picture and speed** section shows the real frame rate under **Frame rate**, and in the corner of the video. To get more frames per second, try these in order:

1. **Resolution:** **640 × 480** is a good balance. **800 × 600** and above are noticeably slower.
2. **Picture quality:** move the slider towards **Smoother**. A value of about 15–20 makes smaller frames that send faster.
3. **Signal:** move the camera closer to the router. Below about -70 dBm the stream slows down a lot.
4. **Viewers:** close extra viewers. Every open page or CCTV app shares the same camera Wi-Fi.
5. **Frame rate cap:** use **Frame rate** to set a limit (for example 10 per second) if you want steadier video or less Wi-Fi traffic. **As fast as possible** removes the limit.

Typical results on a good signal: about 20–25 frames per second at 320 × 240, 12–20 at 640 × 480, and 5–10 at 1280 × 720.

## Face watch

Face watch spots chosen people, flashes the camera light yellow, and records video of them to the device you're watching on.

**Where it runs:**
- **On the camera:** the watch list is stored here, so every phone, tablet or computer sees the same people.
- **On the viewing device:** recognition and recording run in the browser on the device that has the camera page open. The ESP32 alone is too slow for reliable face recognition.

**Add a person**
1. Open the camera page at `http://<ip>/` and switch to the **Faces** tab. It has its own live picture.
2. Under **Add a person**, type their name.
3. With them facing the camera in good light, tap **Take face sample** 3 times. Have them turn their head slightly between samples.
4. Tap **Save person**. Up to 10 people can be stored.

**Start watching**
1. On the **Camera** tab, tap **Start watching**. The first time, the device downloads the face recognition files, about 7 MB, so it needs internet access.
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

## Recording to the microSD card

The board's microSD slot is used in 1-bit mode on **GPIO 38 (CMD), 39 (CLK) and 40 (D0)**.

### Format the card as FAT32 first

Cards over 32 GB come formatted as **exFAT**, which the ESP32 can't read. Formatting erases the card.

**On a Mac**
1. Open **Disk Utility** and choose **View > Show All Devices**.
2. Select the card itself (the top-level device, not the volume under it).
3. Click **Erase**.
4. Set **Format** to **MS-DOS (FAT)** and **Scheme** to **Master Boot Record**.
5. Click **Erase**.

**On Windows**
1. Windows only offers FAT32 for cards up to 32 GB.
2. For bigger cards, use the free **SD Memory Card Formatter** from the SD Association, or a FAT32 formatting tool such as guiformat.

Put the card in the camera **before** powering it on. The camera checks for the card at startup.

### What gets saved

| Folder | Contents |
|---|---|
| `/rec` | Face watch videos, e.g. `20261026_183015_Sam.avi` (date, time, first name seen). Recorded at 10 frames per second and split into 10-minute files |
| `/photos` | Photos from **Save a photo to the card**, e.g. `20261026_184500_photo.jpg` |

- **When it records:** card recording runs while face watch sees someone on your list, and stops when the alert ends. Face watch must be running on a device with the camera page open.
- **The page's controls:** **Record on the card when someone on the list is seen** turns card recording on or off. **Record on this device** does the same for the viewing device, so you can use either or both.
- **Free space:** when less than 2 GB is left, the oldest files are deleted automatically.
- **Date and time:** times are UK time, set from the internet once the camera is on Wi-Fi. Files made before the clock is set start with `notime_`.
- **Playing the files:** AVI files play in VLC, Windows Media Player, QuickTime/macOS and most Android video players.
- **Getting the files:** download them from the **Camera memory card** section of the camera page, or take the card out and use a card reader.

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
| "No memory card found" | Format the card as **FAT32** (not exFAT), insert it with the camera unplugged, then power on. |
| Card recordings don't appear | Check **Record on the card when someone on the list is seen** is ticked, and that face watch is running on a device with the camera page open. |
| A downloaded AVI won't play | Use VLC. Files from a recording cut off by unplugging the camera may be incomplete. |
| Live view freezes while downloading a big file | Normal: the camera serves one download at a time. It resumes when the download finishes. |
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
firmware/esp32s3_cctv/esp32s3_cctv.ino   ESP32-S3 firmware
firmware/esp32s3_cctv/web_page.h        Camera page (live view, face watch, settings)
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
