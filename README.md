# 🔍 Port Scan Early Warning System

> 🚀 A small project to **detect potential port-scanning activities** from large-scale network scan datasets.
> 💡 The dataset used here is from **Masscan Internet Scan (IEEE DataPort / Kaggle)** — formatted as huge JSON files (~9GB).
> 🧪 All processing and implementation are done with **Python** (streaming, memory-safe, and step-by-step).

---

## 🧬 Overview

This project demonstrates a **complete mini-pipeline** from raw network data to actionable security alerts:

1. 🧹 **Data Cleaning & Preprocessing** — Convert large-scale JSON to clean CSV (streaming).
2. ⚙️ **Implementation / Detection** — Analyze traffic patterns, identify port-scan behavior.
3. 📊 **Result & Conclusion** — Evaluate findings, discuss limitations, and future improvements.

---

# 🧹 Step 1 — Clean Data (Preprocessing)

> Dataset: `scan-04_27_2021-1619562631-C17RqCX0.json` (~9GB)

---

## 🎯 Goal

Transform raw **Masscan JSON** (array-based) into a structured **CSV dataset** that can be processed efficiently for later analysis and detection.
Each CSV row represents one `(ip, timestamp, port)` record.

---

## ⚙️ Tools & Libraries

* **Python 3.x**
* **Libraries:** `ijson`, `tqdm`, `pandas` *(optional for verification)*

Install once:

```bash
pip install ijson tqdm pandas
```

---

## 🧠 JSON Structure Overview

Each element in the JSON array looks like:

```json
{
  "ip": "165.221.32.138",
  "timestamp": "1619562631",
  "ports": [
    { "port": 21, "proto": "tcp", "status": "open", "reason": "syn-ack", "ttl": 245 }
  ]
}
```

Because the file is an **array of millions of objects**, we cannot load it fully into memory. Instead, we use **streaming** to iterate over each item using `ijson`.

---

## 🧩 Script: Convert JSON → CSV (Streaming)

File: `D:\json_array_to_csv_sample.py`

```python
import ijson, csv
from tqdm import tqdm

INPUT = r"D:\\scan-04_27_2021-1619562631-C17RqCX0.json"
OUTPUT = r"D:\\sample_clean_portscan.csv"
MAX_ITEMS = 100000   # Process sample of 100k top-level JSON objects

FIELDS = ["ip", "timestamp", "port", "proto", "status", "reason", "ttl"]

def safe_get(d, key):
    return d.get(key) if isinstance(d, dict) else None

def main():
    written = 0
    items_seen = 0
    with open(INPUT, "rb") as fin, open(OUTPUT, "w", newline='', encoding="utf-8") as fout:
        writer = csv.writer(fout)
        writer.writerow(FIELDS)
        for obj in tqdm(ijson.items(fin, "item"), desc="Sampling JSON items"):
            items_seen += 1
            if not isinstance(obj, dict):
                if items_seen >= MAX_ITEMS:
                    break
                continue
            ip = safe_get(obj, "ip")
            ts = safe_get(obj, "timestamp")
            ports = safe_get(obj, "ports")
            if isinstance(ports, list):
                for p in ports:
                    port = safe_get(p, "port")
                    proto = safe_get(p, "proto")
                    status = safe_get(p, "status")
                    reason = safe_get(p, "reason")
                    ttl = safe_get(p, "ttl")
                    writer.writerow([ip, ts, port, proto, status, reason, ttl])
                    written += 1
            if items_seen >= MAX_ITEMS:
                break
    print(f"\n✅ Sample done. Processed {items_seen} JSON objects, wrote {written} CSV rows to {OUTPUT}")

if __name__ == "__main__":
    main()
```

---

## 🧮 Output Example

```text
ip,timestamp,port,proto,status,reason,ttl
165.221.32.138,1619562631,21,tcp,open,syn-ack,245
165.2.102.63,1619562631,8443,tcp,open,syn-ack,55
104.131.104.219,1619562632,80,tcp,open,syn-ack,49
3.131.98.233,1619562632,22,tcp,open,syn-ack,33
...
```

---

## 🧰 Optional Data Filtering

To reduce output size, you can add:

```python
import ipaddress
if ip and ipaddress.ip_address(ip).is_private:
    continue  # skip private IPs
```

Or only keep specific ports:

```python
KEEP_PORTS = {22, 80, 443, 8080, 3389}
if int(port) not in KEEP_PORTS:
    continue
```

---

## 📊 Verification

Use pandas to quickly verify the sample:

```python
import pandas as pd
df = pd.read_csv(r"D:\\sample_clean_portscan.csv")
print(df.head())
print(df['port'].value_counts().head(10))
```

---

## 💾 Output Files

| File                           | Description                                   |
| ------------------------------ | --------------------------------------------- |
| `D:\sample_clean_portscan.csv` | Sample cleaned dataset (CSV)                  |
| `D:\clean_portscan.csv`        | Full cleaned dataset (if processed all items) |

---

## 🧭 Summary

* ✅ Verified file structure → JSON Array.
* ✅ Used **streaming parse** (`ijson`) to handle large-scale data.
* ✅ Converted nested structure into flat table.
* ✅ Created CSV ready for next implementation step (detection).

> ⏭️ Next Step: [Implementation (Detection System)](#)


## ⚙️ Step 2 — Implementation (Detection System)


## 📊 Step 3 — Result & Conclusion


## 🧭 Quick Navigation

| Section                                                        | Description                  |
| -------------------------------------------------------------- | ---------------------------- |
| [🧹 Clean Data](#-step-1--clean-data-preprocessing)            | Convert & preprocess dataset |
| [⚙️ Implementation](#-step-2--implementation-detection-system) | Detection algorithm          |
| [📊 Result](#-step-3--result--conclusion)                      | Findings & conclusions       |

---


Made by *Nguyễn Nhất Duy*
