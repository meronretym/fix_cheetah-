# Cheetah SPI on Raspberry Pi 5 (USB)

This repository documents the fix path for using a **Cheetah SPI Host Adapter** with a **Raspberry Pi 5** over USB.

## Important clarification
The Cheetah adapter is a **USB SPI host** device. It does **not** use the Raspberry Pi `spidev` pins directly, so enabling Pi SPI (`raspi-config`) is not required for Cheetah itself.

## 1) Verify USB detection
```bash
lsusb
```
You should see a Cheetah/Total Phase USB device in the list.

## 2) Install required USB access package
```bash
sudo apt update
sudo apt install -y libusb-1.0-0
```

## 3) Add udev permissions for non-root access
Create:
`/etc/udev/rules.d/99-cheetah.rules`

With:
```udev
SUBSYSTEM=="usb", ATTR{idVendor}=="<VENDOR_ID>", ATTR{idProduct}=="<PRODUCT_ID>", MODE="0664", GROUP="plugdev"
# Example format only: ATTR{idVendor}=="1a2b", ATTR{idProduct}=="3c4d"
```

Get IDs from `lsusb` (the hexadecimal `vvvv:pppp` pair, for example `1a2b:3c4d`).  
Use the IDs reported by your own adapter (do not copy example values).
Ensure your user is in `plugdev`:
```bash
sudo usermod -aG plugdev $USER
```

Then reload rules:
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Unplug/replug the Cheetah adapter.

## 4) Run your Cheetah software/SDK as normal user
If your app still fails to open the adapter, test once with `sudo` to confirm it is a permissions issue. If `sudo` works and normal user does not, the udev rule still needs correction.

## 5) Quick troubleshooting checklist
- Bad/charge-only USB cable -> use a known data cable.
- USB hub power issue -> connect directly to Pi 5 USB port.
- Missing runtime library -> ensure `libusb-1.0-0` is installed.
- Permission denied -> fix udev rule and replug device.
- Multiple adapters -> confirm your software opens the expected adapter index/serial.

## Validation commands
```bash
lsusb
```

Then run `udevadm info -a -n /dev/bus/usb/<BUS_ID>/<DEVICE_ID>` using values from the matching `lsusb` entry (`Bus <BUS_ID> Device <DEVICE_ID>:`).  
`<BUS_ID>` and `<DEVICE_ID>` are zero-padded to three digits (for example `001` and `005`).  
Example: `Bus 001 Device 005` -> `udevadm info -a -n /dev/bus/usb/001/005`.

If needed, share `lsusb` output and your exact Cheetah SDK/API error to narrow the issue further.
