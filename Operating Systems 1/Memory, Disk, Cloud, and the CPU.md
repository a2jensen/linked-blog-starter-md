TLDR : 
- **RAM**: Temporary space for running apps and holding active data (cloud or disk).
- **Disk**: Stores apps/files locally, survives shutdown.
- **Cloud**: Remote data storage accessed by apps.
- **CPU**: Executes instructions, reads data from RAM, not directly from cloud.

- [[#1. RAM vs Disk vs Cloud]]
	- [[#RAM (Main Memory)]], [[#Disk (Local Storage)]], [[#Cloud (Remote Storage)]]
- [[#2. CPU and Data Access]]
- [[#3. Examples]]
	- [[#Opening Instagram]], [[#Playing a Game]]
- [[#4. Key Terminology Summary]]

## 1. RAM vs Disk vs Cloud

### RAM (Main Memory)
- Refers to **DRAM**, the physical memory used by the system to store active data.
- Temporary and **volatile** — wiped on shutdown.
- Holds:
  - App code being executed.
  - Open files (e.g., current Obsidian note).
  - Fetched cloud data (e.g., Instagram feed).
- Data in RAM can come from:
  - **Disk** (local files)
  - **Cloud** (fetched via app requests)

### Disk (Local Storage)
- Persistent, long-term storage on SSD/HDD.
- Stores:
  - Installed applications (e.g., games, Obsidian).
  - Personal files (e.g., documents, media).
- "Not in the cloud" = saved **on disk**.

### Cloud (Remote Storage)
- Data hosted on remote servers (e.g., AWS, Google Cloud).
- Stores:
  - User profiles, media (e.g., Instagram photos).
  - Game save data, settings (e.g., Fortnite character).
- Accessed over the internet by apps via HTTP requests.
- Data fetched from the cloud is stored **temporarily in RAM** while used.

---

## 2. CPU and Data Access

- The **CPU** runs your programs by executing instructions loaded into RAM.
- When data is needed:
  1. CPU first checks **cache** (L1 → L2 → L3)
  2. If not in cache, it reads from **RAM**
  3. If not in RAM (e.g., due to page fault), the OS fetches it from **disk**
  4. Apps may also fetch data from the **cloud**, then place it in RAM

- CPU never directly fetches from the cloud — it executes app code that requests cloud data, which then gets processed locally.

---

## 3. Examples

### Opening Instagram:
- App binary is loaded from **disk to RAM**
- CPU starts running the app
- App fetches user feed from **cloud**
- Feed data is stored in **RAM**, used to render UI
- Some data may be cached to **disk**

### Playing a Game:
- Game assets are on **disk**
- When playing, assets and code are loaded into **RAM**
- **Cloud** stores:
  - Progress
  - Character data
  - Multiplayer states

---

## 4. Key Terminology Summary

| Term        | Meaning                           |
|-------------|-----------------------------------|
| RAM         | Main memory, fast, volatile       |
| DRAM        | Type of RAM used for main memory  |
| Disk        | Local, permanent storage          |
| Cloud       | Remote storage via internet       |
| Cache       | Very fast memory near the CPU     |
| CPU         | Executes instructions from RAM    |

---
