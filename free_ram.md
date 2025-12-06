To **free system buffer/cache (RAM cache)** in Linux, you can safely clear different types of caches using the kernel’s `drop_caches` mechanism.

# 🔧 **1. Check Current Memory Usage**

```bash
free -h
```

---

# 🧹 **2. Clear PageCache Only**

```bash
sudo sync; echo 1 | sudo tee /proc/sys/vm/drop_caches
```

---

# 🧹 **3. Clear dentries and inodes**

```bash
sudo sync; echo 2 | sudo tee /proc/sys/vm/drop_caches
```

---

# 🧹 **4. Clear EVERYTHING (PageCache + dentries + inodes)**

➡️ *Most commonly used*

```bash
sudo sync; echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

# 🔄 **5. Make it Automatic (Optional)**

### Run every hour (cron):

```bash
sudo crontab -e
```

Add:

```
0 * * * * sync; echo 3 > /proc/sys/vm/drop_caches
```

---

# 🛑 **Important Notes**

* Linux automatically manages caches — clearing them frequently **is not recommended**.
* Only use this when:

  * System is under memory pressure.
  * Testing performance.
  * After large file operations.
  * Before restarting heavy apps.

---

# 📌 **To Reduce Swap Pressure**

```bash
sudo sysctl -w vm.swappiness=10
```

(Permanent: add `vm.swappiness=10` to `/etc/sysctl.conf`)

---
## A Ready Script
Make a script `sudo vi clear_cache.sh` in your location
```bash
#!/bin/bash

echo "-----------------------------------------"
echo "   Linux RAM Cache Clearing Utility"
echo "-----------------------------------------"
echo ""
echo "[+] Memory usage BEFORE clearing:"
free -h

echo ""
echo "[+] Syncing disks..."
sync

echo "[+] Dropping caches..."
echo 3 > /proc/sys/vm/drop_caches

echo ""
echo "[+] Memory usage AFTER clearing:"
free -h

echo ""
echo "[✓] Cache cleared successfully."
echo "-----------------------------------------"
```
▶️ Run it anytime   `sudo clear_cache.sh`
