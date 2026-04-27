#  Isolated KVM Networking: The Dedicated Ethernet Bridge
> **Scenario:** Give a Windows VM a direct physical line to the internet via Ethernet while keeping the Linux Host isolated on WiFi.

---

##  The Micro-Logic (The Hardware Flow)
| Component | Logical Role | The "What For" |
| :--- | :--- | :--- |
| **`br0`** | Virtual Switch | A power strip in RAM. It doesn't have an IP; it just passes traffic. |
| **`enp7s0`** | Physical Plug | The "Uplink." The physical wire plugged into the wall router. |
| **`vnet0`** | Virtual Cable | The ghost wire that appears when the VM starts to plug into `br0`. |

---

## Step 1: The Linux Host Setup
Follow these steps to build the "Switchboard" and ensure the Host OS doesn't "steal" the connection.

### 1. Building the "Switch"
```bash
sudo nmcli con add type bridge ifname br0 con-name br0
```
* **Why:** Think of this as unboxing a physical 5-port network switch. Right now, it's just sitting on your desk (RAM) and isn't plugged into anything yet.

### 2. Connecting the Switch to the Real World
```bash
sudo nmcli con add type bridge-slave ifname enp7s0 con-name br0-port master br0
```
* **Why:** You are taking an Ethernet cable and plugging one end into your Laptop's Port (`enp7s0`) and the other into your New Virtual Switch (`br0`).

### 3. Silencing the host (The Isolation)
```bash
sudo nmcli con modify br0 ipv4.method disabled
```
* **Why:** You have to tell the host : "Don't use this switch to get an IP for yourself." This leaves the entire "pipe" open for the VM.

---

##  Step 2: Connecting the VM to the Switch
1. Open **Virt-Manager** -> VM Settings.
2. Select the **NIC (Network Interface)**.
3. Set **Network Source** to `Bridge device...`.
4. Type **`br0`** into the Device Name field.
* **The Logic:** This is like taking a virtual Ethernet cable from your Windows 11 VM and plugging it into a spare port on your Virtual Switch (`br0`).

---

## Step 3: The Windows Driver "An easy way"

### How to install via Folder/USB:
1. Copy the `drivers/` folder from this repo to a USB drive and connect it to the VM to take the files from it
2. Open **Device Manager** in Windows.
3. Right-click the **Ethernet Controller** (with the yellow warning sign).
4. Select **Update Driver** -> **Browse my computer for drivers**.
5. Point it to the `drivers/NetKVM/w11/amd64` folder.
6. Repeat for the **Display Adapter** using the `drivers/qxldod/w11/amd64` folder.

---

## ⚠️ Troubleshooting & Common Errors

###  No Internet Flow
Run the "Switchboard Check" in your Linux terminal:
```bash
bridge link
```
* **The Check:** You should see both `enp7s0` and `vnet0` listed with `master br0 state forwarding`. If you see this, the "electricity" is flowing.

---

##  The Golden Rule: The WiFi Trap
**Never attempt to bridge a WiFi card (`wlp0s20f3`).**
WiFi is a "secure tunnel" that only allows **one** MAC address at a time. If the VM tries to send its identity through your WiFi card, the router assumes it's a hack and kills the connection. **Always use a cable for bridging!**

---

### NOTE;
(`wlp0s20f3`) and (`enp7s0`) are MY interfaces it will ofc be different for u 
