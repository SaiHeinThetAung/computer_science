

---

### First, the **main goal** of your project:
✅ **Read GPS data** from two GPS receivers  
✅ **Calculate heading** (direction) of the ship  
✅ **Save logs** locally in CSV format  
✅ **Send data** to a remote server  
✅ **Retry** sending if the server isn't reachable  
✅ **Run automatically** as a service when Raspberry Pi boots

---

# ⚙ How to Run It

### 1. **Check the Directory**
All files should be placed under:
```
/home/pi/navbox/
```
You should have:
- `main.py`
- `heading_calc.py`
- `gps_logger.py`
- `config.json`
- `retry_queue.json` (optional — it gets created when needed)
- `navbox.service` (service file for auto-start)

---

### 2. **Install Required Python Libraries**
SSH into your Raspberry Pi, then install the needed libraries:
```bash
sudo apt update
sudo apt install python3-pip
pip3 install pyserial requests
```
✅ `serial` comes from `pyserial`
✅ `requests` is for sending HTTP POST

---

### 3. **Make sure your GPS devices are connected**
Plug your two GPS devices into USB ports.

Check they appear:
```bash
ls /dev/ttyUSB*
```
You should see `/dev/ttyUSB0`, `/dev/ttyUSB1` (matching `gps_port_a` and `gps_port_b` in `config.json`).

---

### 4. **Test Manually First**
Manually run the Python script to make sure it works:
```bash
cd /home/pi/navbox
python3 main.py
```
If GPS works and server accepts the data, you will see logs like:
```
위치 A: (37.774912, 127.000182) / 방향: 152.34°
전송 성공: 200
```
If server fails, it will save to `retry_queue.json`.

---

### 5. **Set Up the Service (Auto Start on Boot)**

Move the `navbox.service` file into systemd:
```bash
sudo cp navbox.service /etc/systemd/system/
```

Reload systemd:
```bash
sudo systemctl daemon-reload
```

Enable the service:
```bash
sudo systemctl enable navbox
```

Start the service:
```bash
sudo systemctl start navbox
```

Check status:
```bash
sudo systemctl status navbox
```
You should see it running!

---

# 🧠 Quick Summary of What Happens
| Step | What Happens |
|:-----|:-------------|
| 1 | Reads GPS from two USB ports |
| 2 | Calculates heading (smooths over last 5) |
| 3 | Saves into a daily CSV |
| 4 | Sends to server |
| 5 | Retries later if network fails |
| 6 | Auto-starts when Pi boots |

---

# 📋 Additional Notes
- If the server URL in `config.json` isn't reachable, it **saves to retry_queue.json** and tries again later.
- Logs are saved **daily** under `/home/pi/navbox/logs/` like `gps_2025-04-29.csv`.
- Older logs are **zipped** automatically.
- It’s designed to **run forever** (`Restart=always` in service file).
- Heading is **smoothed** with the last 5 heading values for better stability (`deque` buffer).


Good question — and very important! 🔥  
Let’s go through **why two GPS units** are needed **on the ship** for your system:

---

# 🛥️ Why Two GPS Units Are Used

### 1. **To Calculate the Ship's Heading (Direction) Without a Compass**
- **One GPS** can tell you **position** (latitude and longitude).
- But **one GPS cannot tell you** which way the ship is facing (North, East, South, West).
- **Two GPS units** placed a **fixed distance apart** (like front and back of the ship) **can** tell the direction by comparing their two positions.

---
### 2. **How Heading Calculation Works**

The `heading_calc.py` does this:

- It gets position from **GPS A** (`lat1`, `lon1`).
- It gets position from **GPS B** (`lat2`, `lon2`).
- It **draws an invisible line** from A ➔ B.
- It **calculates the angle** (heading) of that line **relative to North** using trigonometry (using `atan2()` math).

✅ So even if the ship is **stopped** or **moving slowly**, you still know **exactly which direction** the ship is facing!

---
### 3. **If only one GPS was used**
- You would only know **where** the ship is.
- You would **not know which way** it is pointing **unless it was moving**.
- Even then, heading based on movement is **delayed** and **unstable** at low speed.

---

# 📊 Example:

Imagine two GPS antennas like this on the ship:

```
Front of ship ---> [GPS B]
                    |
                    | Ship structure
                    |
Back of ship  ---> [GPS A]
```

If:
- GPS A = (37.7749, 127.0001)
- GPS B = (37.7751, 127.0003)

Then your code calculates **the angle** between these two points and says:

> Heading = 153° ➔ (South-East)

**Even if the ship is not moving**!

---

# 🧠 In short:
| One GPS | Two GPS |
|:--------|:--------|
| Only tells ship’s position | Tells position **AND** facing direction |
| No heading when stationary | Heading even when not moving |
| Poor heading at low speed | Accurate, real-time heading |

---
# 🚀 Real ships
In real-world ships (even big ones):
- Dual GPS heading is **common** for **navigation systems**, **auto-pilot systems**, and **marine control systems**.
- Sometimes they even use **triple GPS** for redundancy!

---

Good — you're thinking like a professional now.  
**Before trusting your GPS system**, you must **test if your code runs correctly** and **find errors or bugs**.  
I'll show you step-by-step **how you can check**:

---

# 🔵 1. Check if the program is running without crashing

- First, **start your service** (your `.service` file).
- Run:
  ```bash
  sudo systemctl start navbox.service
  ```

- Check if service is active:
  ```bash
  sudo systemctl status navbox.service
  ```

- **If "active (running)"**, basic no-crash ✅.

---

# 🔵 2. Check GPS logs are being saved correctly

- Go to your logs folder:
  ```bash
  cd /home/pi/navbox/logs
  ls
  ```

- You should see a file like:
  ```
  gps_2025-04-29.csv
  ```

- Open and view it:
  ```bash
  cat gps_2025-04-29.csv
  ```

- It should show lines like:
  ```
  timestamp,lat1,lon1,lat2,lon2,heading
  2025-04-29T14:00:00,37.7749,127.0001,37.7750,127.0002,152.34
  ...
  ```

✅ **If file exists and data is saving every second**: good!

---

# 🔵 3. Check zip compression works

- After **1 day passes** (or you simulate a past log), the system should:
  - **Make `.zip` file** for yesterday.
  - **Delete** the `.csv` file for yesterday.

- You can manually check your `/logs/` folder after 1 day:
  ```bash
  ls /home/pi/navbox/logs
  ```

✅ **If .zip file created for old days**: good!

---

# 🔵 4. See program logs (catch hidden errors)

- Your systemd service inherits output.
- To see **full logs (errors, prints, crashes)**:
  ```bash
  journalctl -u navbox.service -f
  ```

- It will show real-time messages if any errors happen, like:
  ```
  File not found
  GPS error
  ValueError in heading calculation
  ```

✅ **If no strange errors** and you see "logging saved", perfect!

---

# 🔵 5. Add safety prints (optional during testing)

In `main.py`, add **temporary** `print()` statements like:

```python
print(f"Saving GPS data: {lat1},{lon1} -> {lat2},{lon2}, heading={heading}")
```
or
```python
print("Compressed yesterday's log successfully.")
```

✅ **You can remove prints after you are confident.**

---

# 🔵 6. Special case testing

Think like this:
- What if **GPS signal lost**?
- What if **disk space full**?
- What if **lat1 == lat2, lon1 == lon2**? (No movement?)

You may want to **simulate those** and see if program **handles** without crash.

✅ **Robust system must survive unexpected errors.**

---

# 🔵 7. Code review (important)

Before production:
- Check your code for **wrong assumptions**.
- Example:
  - `heading_buffer` should not be empty when doing `sum(heading_buffer)`.
    ✅ But in your case, it's filled safely.
  - File path permissions (does `pi` user have `/home/pi/navbox/logs` access?).

✅ Your current code is **basic safe**, no big bugs at first look.

---

# 🔥 In simple words:

| Test | How |
|:--|:--|
| Service runs? | `systemctl status navbox.service` |
| Log file created? | Check `/home/pi/navbox/logs` |
| Data inside file? | `cat` or open CSV |
| Errors happening? | `journalctl -u navbox.service -f` |
| Zip works? | Check after 1 day |
| Unexpected cases? | Test manually |

---

# 📢 Tip
**Always** run in small test first.  
**Example**: 
- Change GPS coordinates manually for fake data.
- Force run `compress_old_logs()` by setting date manually to test zipping.

---

