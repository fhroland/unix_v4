# UNIX Fourth Edition (1973) - Reconstruction & Installation

This repository provides a step-by-step guide and the necessary SIMH configuration to install and run the recently recovered "Utah" UNIX V4 tape (December 2025).

### A Milestone in Computer Archeology

In December 2025, the international computer heritage community achieved a sensation: a long-lost magnetic tape from the **University of Utah**, dating back to 1974, was successfully read and reconstructed. This find represents the **Fourth Edition of UNIX (V4)**—a pivotal moment in the history of computing. 

UNIX V4 marks the historic turning point where the operating system was almost entirely rewritten in the **C programming language**. This transition birthed the era of portable software and laid the foundation for the modern digital world as we know it today.

This repository serves as a practical bridge to this discovery. It documents the path from the raw data to a running system, allowing you to set up a functional UNIX V4 in just a few steps. It is a unique opportunity to explore the very first C-based UNIX and study the origins of modern systems architecture.

## Step 1: Prerequisites & Downloads

Before you can boot UNIX V4, you need the PDP-11 emulator and the reconstructed tape image.

### 1. Install the SIMH Emulator
The **SIMH (History Simulator)** is the industry standard for emulating historic hardware. You specifically need the `pdp11` executable.

* **Linux (Ubuntu/Debian):**
    ```bash
    sudo apt update
    sudo apt install simh
    ```
* **Linux (openSuse):** (I have used)
    ```bash
    sudo zypper install simh
    ```
* **macOS (Homebrew):**
    ```bash
    brew install simh
    ```
* **Windows:**
    Download the latest binaries from the [SIMH GitHub Releases](https://github.com/simh/simh) or use [Open SIMH](https://github.com/open-simh/simh). Ensure `pdp11.exe` is in your PATH or project folder.

### 2. Download the Reconstruction Tape
The "Utah" tape contains the original 1973/74 data. Download the digital tape image (`.tap` format) from the official Archive.org repository.

## Step 2: Defining the Virtual Hardware

To run UNIX V4, we must recreate a specific PDP-11 configuration that matches the requirements of the 1970s kernel. We will use a configuration file (often called `boot.ini` or `pdp11.ini`) to "wire" our virtual computer.

### 1. Create a Blank Disk Image
The simulation needs a physical medium to install UNIX onto. We will emulate a **DEC RK05** disk cartridge (approx. 2.5 MB). 

Run the following command in your terminal to create an empty disk file:
```bash
# This creates a zeroed-out file named disk.rk
dd if=/dev/zero of=disk.rk bs=512 count=4872

(Windows users can simply let SIMH create the file via the att command in the next step, but a pre-created file is more reliable.)

### 2. Create the Configuration File (boot.ini)
Create a text file named boot.ini in your project folder. This file tells the simulator which CPU to use, how much memory to plug in, and where the tape and disk drives are connected.

Copy and paste the following configuration:
```
; --- Hardware Definition for UNIX V4 ---
set cpu 11/45          ; Use the PDP-11/45 CPU (Standard for V4)
set cpu 256K           ; Set memory to maximum 256 KB
set rk0 rk05           ; Define the first disk drive as an RK05
set rp disable         ; Disable unused controllers for a clean boot
set rl disable

; --- Attaching Media ---
attach rk0 disk.rk     ; "Insert" our blank disk into drive 0
attach tm0 analog.tap  ; "Insert" the Utah Tape into the tape drive

; --- Boot Procedure ---
boot tm0               ; Tell the CPU to start from the tape drive
```
**Why these settings?**
  * **CPU 11/45:** UNIX V4 was developed and optimized for the 11/45's memory management unit (MMU).
  * **256K:** While 256 KB sounds tiny today, it was a massive amount of "core memory" in 1973, allowing the kernel to run smoothly.
  * **boot tm0:** This command tells the hardware to read the first 512 bytes of the tape into memory and execute them—this is our "Bootloader."




---

> **Note:** If you find these scripts or instructions helpful, please cite this repository: [github.com/fhroland/unix_v4](https://github.com/fhroland/unix_v4)

## Credits & Sources
* **Technical Guide:** Based on the excellent work at [squoze.net/UNIX/v4/](http://squoze.net/UNIX/v4/)
* **Tape Image:** Recovered by Thalia Archibald et al. (Available at [Archive.org](https://archive.org/details/utah_unix_v4_raw))
* **License:** This project is licensed under the BSD 3-Clause License. Original UNIX source code is subject to the Caldera License.
