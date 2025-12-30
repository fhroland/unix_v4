# UNIX Fourth Edition (1973) - Reconstruction & Installation

This repository provides a step-by-step guide and **ready-to-use SIMH configuration files** to install and run the recently recovered “Utah” UNIX V4 tape (December 2025).

### A Milestone in Computer Archaeology

In December 2025, the international computer heritage community achieved a major breakthrough: a long-lost magnetic tape from the **University of Utah** (~1974) was successfully read and reconstructed. This find represents the **Fourth Edition of UNIX (V4)**—a pivotal moment in the history of computing. 

UNIX V4 marks the historic turning point where the operating system was almost entirely rewritten in the **C programming language**. This transition ushered in the era of portable software and laid the foundation for the modern digital world as we know it today.

This repository serves as a practical bridge to this discovery. It documents the path from the raw data to a running system, allowing you to set up a functional UNIX V4 in just a few steps. It is a unique opportunity to explore the very first C-based UNIX and study the origins of modern systems architecture.

## Step 1: Prerequisites & Downloads

Before you can boot UNIX V4, you need the PDP-11 emulator and the reconstructed tape image.

### 1. Download the Reconstruction Tape
The "Utah" tape contains the original 1973/74 data. Download the digital tape image (`.tap` format) from the official [Archive.org repository](https://archive.org/details/utah_unix_v4_raw).

#### 💡 Pro Tip: Verify the File Type
If you are on **Linux** or **macOS**, you can verify that the downloaded file is indeed a valid emulator tape image. Open your terminal and run:

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
* **Linux (openSUSE):** (tested)
    ```bash
    sudo zypper install simh
    ```
* **macOS (Homebrew):**
    ```bash
    brew install simh
    ```
* **Windows:**

  Running UNIX V4 on Windows can be done in two ways:

    1.  **WSL (Highly Recommended):** This is the fastest and most stable method. Install a Linux distribution via WSL and follow the Linux instructions.
        ```bash
        # Inside WSL (e.g., Ubuntu)
        sudo apt update && sudo apt install simh
        ```
    2.  **Native Windows Binaries:** If you prefer running it natively, you can find the latest pre-compiled binaries here:
        * [SIMH Development Binaries](https://github.com/simh/Development-Binaries)
        * or follow instructions on [Open SIMH GitHub](https://github.com/open-simh/simh)

    *Note: Ensure that the executable is named `simh-pdp11` or `pdp11` depending on the version you download.*

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

## Step 3: The Installation Process (mcopy)
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

## Step 5: Interacting with UNIX V4 :-)

Once you see the `#` prompt, you are logged in as `root`. However, the shell (the original Thompson Shell) behaves very differently from modern terminals like 'bash' or 'zsh'.

### ⚠️ Important: How to correct typing errors
In 1973, there was no "Backspace" key in the modern sense. If you make a mistake while typing a command, the system uses "Line Kill" and "Character Erase" symbols typical for that era:

* **The `#` Key (Erase):** If you type a wrong character, type a `#` immediately after it. The system will treat the character before the `#` as deleted. 
    * *Example:* `lss#` will be interpreted by the system as `ls`.
* **The `@` Key (Kill):** If you have messed up the entire line, type an `@` at the end and press Enter. This tells the system to discard the entire line so you can start over fresh.

### Basic Commands to Try
Since there is no tab-completion or command history, you have to type every command precisely:

* **`ls /bin`**: Lists the essential system binaries.
* **`who`**: Shows who is logged in (it should just show `root`).
* **`date`**: Displays the current system time.
* **`cat /etc/passwd`**: View the system's user file.

### Navigating the Filesystem
One of the most notable differences for modern users is that the command `cd` (Change Directory) does not exist yet. In **UNIX V4**, you must use the full command name:

* **`chdir`**: This is the command to change your working directory.
    ```text
    # chdir /bin
    # chdir /
    ```
* **No `cd` shorthand**: Typing `cd` will result in a "not found" error.
* **No home directory**: Typing `chdir` without an argument will not take you "home" (as there is no `$HOME` variable defined yet); you must always specify the target path.

## Step 6: Compiling a New Kernel

In UNIX V4, there was no `make` utility. System libraries and the kernel were built by manually running compiler commands or shell scripts. 

### 1. Understanding the Source Structure
Navigate to the system source directory:
```text
# chdir /usr/sys
# ls -l
```
You will see some *C-include files* (*.h), two libraries ('lib1' and 'lib2') and three main directories:
* **conf:** Contains the configuration files and the assembly language startup code.
* **ken:** Contains the "Kernel" core (Memory management, scheduling, etc.), named after **Ken Thompson**.
* **dmr:** Contains the device drivers and I/O logic, named after **Dennis Ritchie**.

Since this version is famous for being the first one written in **C**, you should take a look at the system sources in these folders. :-)

### 2. Preparing the Workspace
Before we start, we should clean up any old object files or previous build remains to save precious disk space on our 2.5MB drive:

    ```plaintext
    # rm -f *.o
    # rm -f lib1 lib2
    ```

### 3. Building the Core Library (`lib1`)
First, we move into the 'ken' directory to compile the base of our operating system.

1. **Change Directory:**
    ```plaintext
    # chdir ken
    ```

2. **Create the Library Build Script:** Since we don't have make, we create a shell script named `mklib.sh` using the `ed` editor. This script will compile each `.c` file and replace it in our library archive.
   Create the build script using ed: Since there is no make, we manually create a shell script. Start the editor:
    ```plaintext
    # ed mklib.sh
    ```

   Now, enter the following commands exactly. Type `a` to start appending text, then paste/type the `ar` commands, and finish with a dot `.` on a new line:

   ```plaintext
    a
    ar r ../lib1 main.o
    ar r ../lib1 alloc.o
    ar r ../lib1 iget.o
    ar r ../lib1 prf.o
    ar r ../lib1 rdwri.o
    ar r ../lib1 slp.o
    ar r ../lib1 subr.o
    ar r ../lib1 text.o
    ar r ../lib1 trap.o
    ar r ../lib1 sig.o
    ar r ../lib1 sysent.o
    ar r ../lib1 sys1.o
    ar r ../lib1 sys2.o
    ar r ../lib1 sys3.o
    ar r ../lib1 sys4.o
    ar r ../lib1 nami.o
    ar r ../lib1 fio.o
    ar r ../lib1 clock.o
    .
    w
    q
    ```
    
    _Note: After typing `w`, `ed` will display the number of bytes written to the file._

4. **Compile the kernel sources:** Before running the script, we must compile the C files into object files (`.o`):
    ```plaintext
    # cc -c *.c
    ```

5. **Run the archive script:** Now, execute the script to bundle the object files into `lib1`:
    ```plaintext
    # sh mklib.sh
    ```
   
### 💡 A Note on Script Execution
You might notice that we executed the script using `sh mklib.sh` instead of making the file executable. In UNIX V4, the "executable bit" for scripts was not yet a standard convention as it is today. By passing the file as an argument to `sh` (the Thompson Shell), we are explicitly telling the shell to read and execute the commands inside the file, regardless of its permissions.

5. **Cleanup:** Storage is extremely limited on an **RK05** disk. Delete the object files immediately after they are added to the library:
    ```plaintext
    # rm -f *.o
    ```

### 4. Building the Driver Library (`lib2`)
Now we repeat the process for the device drivers located in the `dmr` directory. These files handle the communication with the hardware, such as the disk drives and the console.

1. **Navigate to the directory:**
    ```text
    # chdir ../dmr
    ```
   
2. **Compile the driver sources:**
    ```plaintext
    # cc -c *.c
    ```
   
3. **Archive into `lib2`:** We can follow the same steps described above to create a shell script 'mklib.sh' for the following files:
    ```plaintext
    ar r ../lib2 bio.o
    ar r ../lib2 tty.o
    ar r ../lib2 malloc.o
    ar r ../lib2 pipe.o
    ar r ../lib2 cat.o
    ar r ../lib2 dc.o
    ar r ../lib2 dn.o
    ar r ../lib2 dc.o
    ar r ../lib2 dn.o
    ar r ../lib2 dp.o
    ar r ../lib2 kl.o
    ar r ../lib2 mem.o
    ar r ../lib2 pc.o
    ar r ../lib2 rf.o
    ar r ../lib2 rk.o
    ar r ../lib2 tc.o
    ar r ../lib2 tm.o
    ar r ../lib2 vs.o
    ar r ../lib2 vt.o
    ar r ../lib2 partab.o
    ar r ../lib2 rp.o
    ar r ../lib2 lp.o
    ar r ../lib2 dhdm.o
    ar r ../lib2 dh.o
    ar r ../lib2 dhfdm.o
    ```

    or instead of using a script to insert selected drivers into the library, we can also run the archiver directly with a wildcard to save time, as we want to include all compiled drivers into the second library `lib2`:
    
    ```plaintext
    # ar r ../lib2 *.o
    ```

4. **Cleanup:** Again, remove the object files to keep the disk from filling up:
    ```plaintext
    # rm -f *.o
    ```

### 5. Final Assembly and Configuration (`conf`)
The `conf` directory contains the "glue" that connects the kernel to the hardware. Here we find the assembly startup code and the configuration table.

1. **Navigate to the directory:**
   ```text
   # chdir ../conf
   ```
2. **Assemble the Low-Level Code:** We use the assembler (`as`) to process the trap vectors and the machine-language startup code. Note that the output of the assembler is always named `a.out` by default, so we must rename it immediately:
    ```plaintext
    # as low.s
    # mv a.out low.o
    # as mch.s
    # mv a.out mch.o
    ```
3. **Compile the Configuration Table:** The file `conf.c` defines which device drivers from `lib2` are actually included in the kernel.
    ```plaintext
    # cc -c conf.c
    ```
    
6. **Linking the New Kernel:**
    Now we have all the pieces ready: `low.o`, `mch.o`, `conf.o`, and our two libraries `lib1` and `lib2`. We use the linker (`ld`) to combine them into one executable file.

7. **The Final Link:**
    ```plaintext
    # ld -a low.o mch.o conf.o ../lib1 ../lib2
    ```
    
    Note: The `-a` flag tells the linker to create an absolute executable. The resulting file is named `a.out`.

8. **Install the Kernel:** Copy the finished kernel to the root directory and give it a name:
    ```plaintext
    # cp a.out /unixv4
    ```
    
9. **Safe Shutdown:** Before testing your new kernel, ensure all data is written to the disk image:
    ```plaintext
    # sync
    # sync
    # sync
    ```
## Step 7: Testing Your New Kernel
To boot your freshly compiled kernel, restart the simulator. When the '@' prompt appears, instead of typing 'unix', type your new filename:
    ```plaintext
    @unixv4
    ```
If you see the `mem = ...` message and the `login` prompt, you have successfully compiled UNIX Fourth Edition from source! 
It's June 1973 and you are working with the first C-based operating system - UNIX V4. :-) 
   
---
---

> **Note:** If you find these scripts or instructions helpful, please cite this repository: [github.com/fhroland/unix_v4](https://github.com/fhroland/unix_v4)

## Credits & Sources
* **Technical Guide:** Based on the excellent work at [squoze.net/UNIX/v4/](http://squoze.net/UNIX/v4/)
* **Tape Image:** Recovered by Thalia Archibald et al. (Available at [Archive.org](https://archive.org/details/utah_unix_v4_raw))
* **License:** This project is licensed under the BSD 3-Clause License. Original UNIX source code is subject to the Caldera License.
