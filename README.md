# Audio Experiments

This repository contains various experiments and configurations related to audio setups, particularly focusing on Linux environments.

The goal is to document and share successful configurations for different audio hardware and software combinations.

## Behringer UM2 Setup on Fedora 42 (PipeWire + WirePlumber)

### 🧭 Step 1 — Confirm your UM2 node names

From your logs, the relevant names are:

```
alsa_input.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-input
alsa_output.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-output
```

Those are what we’ll use to make it default.

---

### ⚙️ Step 2 — Create a WirePlumber rule

Create the config file:

```bash
mkdir -p ~/.config/wireplumber/main.lua.d
nano ~/.config/wireplumber/main.lua.d/51-um2-default.lua
```

Paste this inside:

```lua
-- Set Behringer UM2 as default for both input and output
alsa_monitor.rules = {
  {
    matches = {
      { { "node.name", "equals", "alsa_input.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-input" }, },
      { { "node.name", "equals", "alsa_output.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-output" }, },
    },
    apply_properties = {
      ["node.description"] = "Behringer UM2",
      ["priority.session"] = 200,
      ["priority.driver"] = 200,
    },
  },
}

-- Force default to the UM2
default_nodes = {
  ["Audio/Source"] = "alsa_input.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-input",
  ["Audio/Sink"] = "alsa_output.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-output",
}
```

Save and exit.

---

### 🔁 Step 3 — Restart WirePlumber and PipeWire

```bash
systemctl --user restart wireplumber pipewire pipewire-pulse
```

---

### 🧪 Step 4 — Verify defaults

```bash
pactl info | grep "Default"
```

Expected output:

```
Default Sink: alsa_output.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-output
Default Source: alsa_input.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-input
```

---

### 🎧 Step 5 — Quick Test

Record:

```bash
arecord -D alsa_input.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-input -f cd -t wav test.wav
```

Play back:

```bash
aplay -D alsa_output.usb-Burr-Brown_from_TI_USB_Audio_CODEC-00.analog-stereo-output test.wav
```

---

### 🧰 Step 6 — GUI Verification (Optional)

Install and open `pavucontrol`:

```bash
sudo dnf install pavucontrol
pavucontrol
```

Check **Input Devices** and **Output Devices** tabs for “Behringer UM2” as fallback.

---

### 🧠 Step 7 — Works for Recording & Daily Use

- Desktop apps → PulseAudio layer (bridged to PipeWire)
- DAWs → JACK layer (bridged to PipeWire)
- No manual switching needed

✅ Recording ready  
✅ Daily playback ready  
✅ Auto-reconnects when plugged/unplugged

---

### ⚡ Optional: Low-Latency Tuning

Create file:

```bash
mkdir -p ~/.config/pipewire/pipewire.conf.d
nano ~/.config/pipewire/pipewire.conf.d/10-low-latency.conf
```

Add:

```ini
context.properties = {
    default.clock.rate          = 48000
    default.clock.allowed-rates = [ 44100, 48000 ]
    default.clock.quantum       = 128
    default.clock.min-quantum   = 64
    default.clock.max-quantum   = 512
    default.clock.quantum-limit = 1024
    default.clock.n-threads     = 2
}
```

Then restart PipeWire:

```bash
systemctl --user restart pipewire
```

