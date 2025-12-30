# UNIX Fourth Edition (1973) - Reconstruction & Installation

This repository provides a step-by-step guide and the necessary SIMH configuration to install and run the recently recovered "Utah" UNIX V4 tape (December 2025).

### A Milestone in Computer Archeology

In December 2025, the international computer heritage community achieved a sensation: a long-lost magnetic tape from the **University of Utah**, dating back to 1974, was successfully read and reconstructed. This find represents the **Fourth Edition of UNIX (V4)**—a pivotal moment in the history of computing. 

UNIX V4 marks the historic turning point where the operating system was almost entirely rewritten in the **C programming language**. This transition birthed the era of portable software and laid the foundation for the modern digital world as we know it today.

This repository serves as a practical bridge to this discovery. It documents the path from the raw data to a running system, allowing you to set up a functional UNIX V4 in just a few steps. It is a unique opportunity to explore the very first C-based UNIX and study the origins of modern systems architecture.

## Step 1: Prerequisites & Downloads

Before you can boot UNIX V4, you need the PDP-11 emulator and the reconstructed tape image.

### 1. Download the Reconstruction Tape
The "Utah" tape contains the original 1973/74 data. Download the digital tape image (`.tap` format) from the official Archive.org repository.

#### 💡 Pro Tip: Verify the File Type
If you are on Linux or macOS, you can verify that the downloaded file is indeed a valid emulator tape image. Open your terminal and run:

```bash
file analog.tap
```
**Expected Output:**

```Plaintext
analog.tap: SIMH tape data
```

This confirmation ensures that the file wasn't corrupted during download and is correctly recognized as a SIMH tape data container, which the PDP-11 simulator can "mount" as a physical magnetic tape.

### 2. Install the SIMH Emulator
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



## Step 2: Defining the Virtual Hardware

To run UNIX V4, we must recreate a specific PDP-11 configuration that matches the requirements of the 1970s kernel. We will use a configuration file (often called `boot.ini` or `pdp11.ini`) to "wire" our virtual computer.

### 1. Create a Blank Disk Image
The simulation needs a physical medium to install UNIX onto. We will emulate a **DEC RK05** disk cartridge (approx. 2.5 MB). 

Run the following command in your terminal to create an empty disk file:
```bash
# This creates a zeroed-out file named disk.rk
dd if=/dev/zero of=disk.rk bs=512 count=4872
```
alternatively you could also use the `truncate` command to create a "sparse file":

```bash
truncate -s 2500k disk.rk
```

(Windows users can simply let SIMH create the file via the att command in the next step, but a pre-created file is more reliable.)

### 2. Create the Configuration File (boot.ini)

Create a text file named boot.ini in your project folder. This file tells the simulator which CPU to use, how much memory to plug in, and where the tape and disk drives are connected.

```text
; --- Hardware Definition for UNIX V4 ---
set cpu 11/45          ; Use the PDP-11/45 CPU
set cpu 256K           ; Set memory to 256 KB
; --- System State ---
d sr 1                 ; Set Switch Register to 1 (Enables Single-User Mode)
; --- Attaching Media ---
attach rk0 disk.rk     ; Attach our blank disk image
attach tm0 analog.tap  ; Attach the Utah Tape image
; --- Boot Procedure ---
boot -o tm0            ; Boot from the tape drive
```

**Why these settings?**
  * **CPU 11/45:** UNIX V4 was developed and optimized for the 11/45's memory management unit (MMU).
  * **256K:** While 256 KB sounds tiny today, it was a massive amount of "core memory" in 1973, allowing the kernel to run smoothly.
  * **boot tm0:** This command tells the hardware to read the first 512 bytes of the tape into memory and execute them—this is our "Bootloader."

### 3. Launch the Simulator
On many modern Linux systems (like Debian, Ubuntu, openSUSE,...), the simulator is called `simh-pdp11`. Run the following command to start the installation:

```bash
simh-pdp11 boot.ini
```
_Note: If you compiled SIMH from source, the command might just be ```pdp11```._

Once the simulator starts, it will read the bootloader from the tape. When the hardware initialization is complete, you will be greeted by the standalone prompt:
```
=
```

## Step 3. The Installation Process (mcopy)
After launching the simulator with `simh-pdp11 boot.ini`, you will see the `=` prompt. This is the standalone bootloader from the tape. Since our `disk.rk` is empty, we must copy the system from the tape to the disk.

Type the following commands (press Enter after each line):

```text
=mcopy
from: k       (This stands for the RK disk controller)
unit: 0       (Our disk.rk is attached to unit 0)
offset: 75    (The Unix V4 data on this specific tape starts at block 75)
count: 4000   (We copy 4000 blocks to ensure the entire system is transferred)
```

## Step 4: First Boot from Disk

Now that the system files have been copied to `disk.rk`, we need to transition from the installation phase to the actual operating system boot.

### 1. Exit and Reconfigure
First, shut down the current simulation session:
* Press `Ctrl+E` to return to the SIMH prompt.
* Type `quit` and press Enter.

### 2. Update the `boot.ini`
Open your `boot.ini` file and change the last line. We no longer want to boot the tape (`tm0`); we want to boot the primary disk (`rk0`). Your final `boot.ini` should look like this:

```text
set cpu 11/45
set cpu 256K
d sr 1
attach rk0 disk.rk
attach tm0 analog.tap
; Boot from the hard disk instead of the tape
boot rk0
```

### 3. Loading the Kernel
Start the simulator again:
```bash
simh-pdp11 boot.ini
```

After the simulator initializes (you might see a message like `Disabling XQ`), the bootloader waits for your input. Unlike modern systems, it requires manual interaction to find the kernel:

* **Press** `k`: This tells the bootloader to look at the **RK05** disk drive.

* **Type** `unix` **and press Enter**: This specifies the filename of the kernel to be loaded.

You should then see the memory initialization message and the system will present the login prompt:

```text
mem = 64530

login:
```

### 4. Logging In
In UNIX V4, you can log in as the superuser by typing:

```text
root
```

_(Note: At this stage, there is typically no password set for the root account.)_




> **Note:** If you find these scripts or instructions helpful, please cite this repository: [github.com/fhroland/unix_v4](https://github.com/fhroland/unix_v4)

## Credits & Sources
* **Technical Guide:** Based on the excellent work at [squoze.net/UNIX/v4/](http://squoze.net/UNIX/v4/)
* **Tape Image:** Recovered by Thalia Archibald et al. (Available at [Archive.org](https://archive.org/details/utah_unix_v4_raw))
* **License:** This project is licensed under the BSD 3-Clause License. Original UNIX source code is subject to the Caldera License.
