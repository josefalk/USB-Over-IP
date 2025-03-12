# USB-Over-IP
 setup usb over ip on raspberry pi as a server, and windows client
# USB Over IP Setup: Raspberry Pi (Server) & Windows (Client)

This guide explains how to set up **USB over IP** using a **Raspberry Pi as a server** and **Windows as a client**. It includes auto-attach scripts for Windows to automatically connect all USB devices on the Pi.

---

## 1. Setup USB/IP on Raspberry Pi (Server)

### **Install Required Packages**
```bash
sudo apt update && sudo apt install usbip-utils linux-tools-generic -y
```

### **Load Kernel Modules**
```bash
sudo modprobe usbip-core
sudo modprobe usbip-host
sudo modprobe vhci-hcd
```
To load them automatically at boot:
```bash
echo -e "usbip-core\nusbip-host\nvhci-hcd" | sudo tee -a /etc/modules
```

### **Start USB/IP Daemon**
```bash
sudo usbipd -D
```

### **List Available USB Devices**
```bash
sudo usbip list -l
```
Example output:
```
Local USB devices
=================
1-1: Logitech USB Receiver (04d9:1203)
```

### **Bind USB Device**
Replace `1-1` with your device ID:
```bash
sudo usbip bind -b 1-1
```

### **Auto-Start USB/IP on Boot**
```bash
sudo nano /etc/systemd/system/usbipd.service
```
Paste:
```ini
[Unit]
Description=USB over IP daemon
After=network.target

[Service]
ExecStart=/usr/sbin/usbipd -D
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```
Save and enable the service:
```bash
sudo systemctl enable usbipd
sudo systemctl start usbipd
```

---

## 2. Install USB/IP on Windows (Client)

### **Install USB/IP Drivers**
1. Download **USB/IP for Windows** from [this GitHub repo](https://github.com/cezanne/usbip-win/releases).
2. Extract files and run `usbip_install_driver.cmd` as **Administrator**.
3. Open **Command Prompt (Admin)** and check if USB/IP is installed:
   ```cmd
   usbip.exe list -r <RaspberryPi_IP>
   ```

### **Attach a USB Device**
```cmd
usbip.exe attach -r <RaspberryPi_IP> -b 1-1
```
Replace `1-1` with the device ID.

### **Detach a USB Device**
```cmd
usbip.exe detach -p <port_number>
```
Find the port number:
```cmd
usbip.exe port
```

---

## 3. Auto-Connect USB Devices on Windows

### **Create a Windows Script to Auto-Connect All USB Devices**
1. Open **Notepad** and paste:
   ```cmd
   @echo off
   set PI_IP=192.168.1.100  REM Change to your Raspberry Pi's IP

   echo Listing USB devices from %PI_IP%...
   for /f "tokens=3" %%i in ('usbip.exe list -r %PI_IP% ^| findstr "busid"') do (
       echo Attaching USB device: %%i
       usbip.exe attach -r %PI_IP% -b %%i
   )
   
   echo All available USB devices attached successfully!
   pause
   ```
2. **Save as `connect_usb.bat`**.
3. Run as **Administrator** to auto-connect all USB devices.

### **Auto-Run Script on Startup**
1. Press `Win + R`, type `taskschd.msc`, and press **Enter**.
2. Click **Create Basic Task** > Name it **USBIP Auto Connect**.
3. Select **"When the computer starts"**.
4. Select **Start a Program**, browse to `connect_usb.bat`.
5. Click **Finish**.

### **Detach All USB Devices (Script)**
```cmd
@echo off
for /f "tokens=3" %%i in ('usbip.exe port ^| findstr "Port"') do (
    echo Detaching USB device on port %%i
    usbip.exe detach -p %%i
)
echo All USB devices detached successfully!
pause
```
Save as **`detach_usb.bat`** and run to disconnect all devices.

---

## 4. Troubleshooting

### **1. Fix "vhci driver is not loaded" on Windows**
```cmd
usbip.exe install vhci
```
If it still fails, manually install **vhci driver**:
1. Open **Device Manager** (`devmgmt.msc`).
2. Click **Action > Add legacy hardware**.
3. Select **Install the hardware that I manually select**.
4. Click **Have Disk** > Browse to `usbip-win-0.3.6-dev\vhci\amd64`.
5. Select `usbip_vhci.inf` and install.

### **2. Check USB Speed on Raspberry Pi**
To check if your camera is running **USB 3.0 (5000M)** or **USB 2.0 (480M)**:
```bash
lsusb -t
```
If your camera shows **480M**, it's running at USB 2.0 speed.
To fix:
```bash
echo 'dtoverlay=dwc3,dr_mode=host' | sudo tee -a /boot/config.txt
sudo reboot
```

### **3. Alternative Methods for Remote Webcam Access**
If USB/IP does not work well for **1080p cameras**, try **MJPEG Streamer**:
```bash
sudo apt install mjpeg-streamer
mjpg_streamer -i "input_uvc.so -r 1920x1080 -f 15" -o "output_http.so -w /usr/share/mjpeg-streamer/www"
```
Access it from Windows via:
```
http://<RaspberryPi_IP>:8080/?action=stream
```

---

## 🎉 Done!
Now your **Raspberry Pi acts as a USB server**, and your **Windows PC auto-connects to USB devices**. 🚀

Let me know if you need help! 😊

