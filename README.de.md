# UNIX V4 „Utah Tape“ (1974) — Installieren & Ausführen mit SIMH (Recovered Dez 2025)

(UNIX Fourth Edition – Rekonstruktion & Installation)

**Schlagwörter:** UNIX V4, Utah tape, University of Utah, 1974, recovered tape, SIMH, PDP-11 emulator

> 🇬🇧 Englische Version dieser Anleitung / English version of this guide: [README.md](README.md)

Dieses Repository bietet eine Schritt-für-Schritt-Anleitung sowie sofort nutzbare SIMH‑Konfigurationsdateien, um das kürzlich wiederhergestellte „Utah“ UNIX‑V4‑Band (Dezember 2025) zu installieren und auszuführen.

## Ein Meilenstein der Computerarchäologie

Im Dezember 2025 gelang der internationalen Computer‑Heritage‑Community ein großer Durchbruch: Ein lange verschollenes Magnetband der University of Utah (Juni 1974) konnte erfolgreich gelesen und rekonstruiert werden. Dieser Fund repräsentiert die **Fourth Edition von UNIX (V4)** – einen Wendepunkt in der Geschichte der Informatik.

UNIX V4 markiert den historischen Moment, in dem das Betriebssystem nahezu vollständig in der **Programmiersprache C** neu geschrieben wurde. Dieser Übergang leitete die Ära portabler Software ein und legte das Fundament für die moderne digitale Welt, wie wir sie heute kennen.

Dieses Repository dient als praktische Brücke zu dieser Entdeckung. Es dokumentiert den Weg von den Rohdaten zu einem laufenden System und ermöglicht es dir, in nur wenigen Schritten ein funktionsfähiges UNIX V4 aufzusetzen. Es ist eine einmalige Gelegenheit, das erste C‑basierte UNIX zu erkunden und die Ursprünge moderner Systemarchitektur zu studieren.

---

## Schritt 1: Voraussetzungen & Downloads

Bevor du UNIX V4 booten kannst, benötigst du den PDP‑11‑Emulator sowie das rekonstruierte Tape‑Image.

### 1. Rekonstruiertes Tape herunterladen

Das „Utah“-Tape enthält die originalen Daten von 1973/74. Lade das digitale Tape‑Image (Format `.tap`) aus dem offiziellen [Archive.org repository](https://archive.org/details/utah_unix_v4_raw) herunter.

#### 💡 Pro‑Tipp: Dateityp prüfen

Unter Linux oder macOS kannst du verifizieren, dass die heruntergeladene Datei tatsächlich ein gültiges Emulator‑Tape‑Image ist. Öffne ein Terminal und führe aus:
```bash
    file analog.tap
```
**Erwartete Ausgabe:**
```plaintext
    analog.tap: SIMH tape data
```

Diese Bestätigung stellt sicher, dass die Datei beim Download nicht beschädigt wurde und als SIMH‑Tape‑Container erkannt wird, den der PDP‑11‑Simulator wie ein physisches Magnetband „mounten“ kann.

### 2. SIMH‑Emulator installieren

**SIMH (History Simulator)** ist der Industriestandard zur Emulation historischer Hardware. Du benötigst konkret das ausführbare Programm `pdp11`.

- **Linux (Ubuntu/Debian):**
    ```bash
      sudo apt update
      sudo apt install simh
    ```
- **Linux (openSUSE) (getestet):**
    ```bash
      sudo zypper install simh
    ```
- **macOS (Homebrew):**
    ```bash
      brew install simh
    ```
- **Windows:**

    UNIX V4 lässt sich unter Windows auf zwei Arten ausführen:

    1. **WSL (sehr empfohlen):** Das ist die schnellste und stabilste Methode. Installiere eine Linux‑Distribution über WSL und folge dann den Linux‑Anweisungen.
    ```bash
        # In WSL (z. B. Ubuntu)
        sudo apt update && sudo apt install simh
    ```
    2. **Native Windows‑Binaries:** Wenn du es nativ ausführen möchtest, findest du die aktuellen vorkompilierten Binaries hier:
   - [SIMH Development Binaries](https://github.com/simh/Development-Binaries)
   - oder folge den Anweisungen im [Open SIMH GitHub](https://github.com/open-simh/simh)

    *Hinweis: Stelle sicher, dass die ausführbare Datei – je nach Version – `simh-pdp11` oder `pdp11` heißt.*

---

## Schritt 2: Virtuelle Hardware definieren

Um UNIX V4 auszuführen, müssen wir eine PDP‑11‑Konfiguration nachbilden, die zu den Anforderungen des 1970er‑Kernels passt. Dazu verwenden wir eine Konfigurationsdatei (oft `boot.ini` oder `pdp11.ini`), die unseren virtuellen Rechner „verkabelt“.

### 1. Leeres Disk‑Image anlegen

Die Simulation braucht ein physisches Medium, auf das UNIX installiert werden kann. Wir emulieren eine **DEC RK05**‑Disk‑Cartridge (ca. 2,5 MB).

Erzeuge ein leeres Disk‑File mit folgendem Befehl:
```bash    
    # Erzeugt eine mit Nullen gefüllte Datei namens disk.rk
    dd if=/dev/zero of=disk.rk bs=512 count=4872
```
Alternativ kannst du auch `truncate` verwenden, um ein „sparse file“ zu erstellen:
```bash
    truncate -s 2500k disk.rk
```
(Windows‑Nutzer*innen können SIMH die Datei auch über den `att`‑Befehl im nächsten Schritt erstellen lassen – ein vorab erzeugtes File ist jedoch zuverlässiger.)

### 2. Konfigurationsdatei erstellen (boot.ini)

Erstelle eine Textdatei namens `boot.ini` in deinem Projektordner. Diese Datei sagt dem Simulator, welche CPU zu verwenden ist, wie viel Speicher vorhanden ist und wo Tape‑ und Disk‑Drives angeschlossen sind.

    ; --- Hardware Definition for UNIX V4 ---
    set cpu 11/45          ; Use the PDP-11/45 CPU
    set cpu 256K           ; Set memory to 256 KB
    ; --- System State ---
    d sr 1                 ; Set Switch Register to 1 (Enables Single-User Mode)
    ; --- Attaching Media ---
    att rk0 disk.rk     ; Attach our blank disk image
    att tm0 analog.tap  ; Attach the Utah Tape image
    ; --- Boot Procedure ---
    boot -o tm0            ; Boot from the tape drive

**Warum diese Einstellungen?**
    - **CPU 11/45:** UNIX V4 wurde für die PDP-11/45 entwickelt und optimiert.
    - **256K:** Heute winzig, 1974 jedoch enorm viel „Core Memory“ – damit läuft der Kernel stabil.
    - **boot tm0:** Lädt die ersten 512 Bytes des Bandes in den Speicher und führt sie aus – unser „Bootloader“.

### 3. Simulator starten

Auf vielen modernen Linux‑Systemen (Debian, Ubuntu, openSUSE, …) heißt der Simulator `simh-pdp11`. Starte die Installation mit:
```bash
    simh-pdp11 boot.ini
```
_Hinweis: Wenn du SIMH aus dem Quellcode kompiliert hast, kann der Befehl auch einfach `pdp11` heißen._

Nach dem Start liest der Simulator den Bootloader vom Tape. Sobald die Hardware‑Initialisierung abgeschlossen ist, erscheint der Standalone‑Prompt:
```plaintext
    =
```

---

## Schritt 3: Installation von Unix V4 (mcopy)

Nach dem Start von `simh-pdp11 boot.ini` siehst du den `=`‑Prompt. Das ist der Standalone‑Bootloader vom Tape. Da `disk.rk` noch leer ist, müssen wir das System vom Tape auf die Disk kopieren.

Gib die folgenden Befehle ein (jeweils Enter nach jeder Zeile):
```plaintext
    =mcopy
    from: k       (Das steht für den RK‑Disk‑Controller)
    unit: 0       (disk.rk hängt an Unit 0)
    offset: 75    (Die UNIX‑V4‑Daten auf diesem Tape beginnen bei Block 75)
    count: 4000   (Wir kopieren 4000 Blöcke, um das gesamte System zu übertragen)
```

---

## Schritt 4: Erster Boot von Disk

Nachdem die Systemdateien nach `disk.rk` kopiert wurden, wechseln wir von der Installationsphase zum eigentlichen Systemboot.

### 1. Beenden und neu konfigurieren

Beende die aktuelle Simulation:
- Drücke `Ctrl+E`, um zum SIMH‑Prompt zurückzukehren.
- Tippe `quit` und drücke `Enter`.

### 2. `boot.ini` anpassen

Öffne `boot.ini` und ändere die letzte Zeile. Wir booten nun nicht mehr vom Tape (`tm0`), sondern von der Disk (`rk0`). Die finale `boot.ini` sollte so aussehen:
```plaintext
    set cpu 11/45
    set cpu 256K
    d sr 1
    attach rk0 disk.rk
    attach tm0 analog.tap   
    ; Boot from the hard disk instead of the tape
    boot rk0
```

### 3. Kernel laden

Starte den Simulator erneut:
```plaintext
    simh-pdp11 boot.ini
```

Nachdem der Simulator initialisiert hat (wahrscheinlich siehst du z. B. `Disabling XQ`), wartet der Bootloader der PDP-11 auf deine Eingabe. Anders als moderne Systeme erfordert er manuelle Interaktion, um den Kernel zu finden:
- **Drücke `k`**: Dadurch schaut der Bootloader auf das RK05‑Disk‑Laufwerk.
- **Tippe** `unix` **und drücke Enter**: Das ist der Dateiname des zu ladenden Kernels.

Danach solltest du die Speicher‑Initialisierung sehen und anschließend den Login‑Prompt:
```plaintext
    mem = 64530

    login:
```

### 4. Einloggen

In UNIX V4 kannst du dich als Superuser anmelden mit:
```plaintext
    root
```

_Hinweis: In diesem Stadium ist typischerweise kein Passwort für `root` gesetzt._

---

## Schritt 5: Mit UNIX V4 arbeiten :-)

Wenn du den `#`‑Prompt siehst, bist du als `root` eingeloggt. Die Shell (die originale Thompson Shell) verhält sich jedoch deutlich anders als moderne Terminals wie `bash` oder `zsh`.

### ⚠️ Wichtig: Tippfehler korrigieren

In den frühen 1970ern gab es keine „Backspace“-Taste im heutigen Sinn. Bei Tippfehlern nutzt das System spezielle Symbole für „Character Erase“ und „Line Kill“:

- **Taste `#` (Erase):** Wenn du ein falsches Zeichen getippt hast, gib sofort ein `#` danach ein. Das System behandelt das Zeichen vor dem `#` als gelöscht.  
  - *Beispiel:* `lss#` wird als `ls` interpretiert.
- **Taste `@` (Kill):** Wenn die ganze Zeile _vermurkst_ ist, tippe am Ende ein `@` und drücke Enter. Dies verwirft die gesamte Zeile und du kannst neu mit der Eingabe beginnen.

### Ein paar Basis‑Kommandos

Da es damals weder Tab‑Completion noch Command‑History gab, musst du alles exakt tippen:

- **`ls /bin`**: Listet die wichtigsten System‑Binaries.
- **`who`**: Zeigt, wer eingeloggt ist (sollte nur `root` anzeigen).
- **`date`**: Zeigt die Systemzeit.
- **`cat /etc/passwd`**: Zeigt die User‑Datei.

### Im Dateisystem navigieren

Eine der auffälligsten Unterschiede: Den Befehl `cd` gibt es noch nicht. In **UNIX V4** musst du den vollständigen Befehlsnamen verwenden:

- **`chdir`**: Wechselt das Arbeitsverzeichnis.
    ```plaintext
      # chdir /bin
      # chdir /
    ```
- **Kein `cd`‑Alias**: `cd` führt zu „not found“.
- **Kein Home‑Directory**: `chdir` ohne Argument bringt dich nicht „nach Hause“ (es gibt noch kein `$HOME`); du musst immer einen Pfad angeben.

---

## Schritt 6: Einen neuen Kernel kompilieren

In UNIX V4 gab es kein `make`. Systembibliotheken und Kernel wurden durch manuelle Compiler‑Aufrufe oder Shell‑Skripte gebaut.

### 1. Quellcode‑Struktur verstehen

Wechsle ins System‑Source‑Verzeichnis:
```plaintext
    # chdir /usr/sys
    # ls -l
```
Du siehst einige *C‑Include‑Files* (`*.h`), zwei Libraries (`lib1` und `lib2`) sowie drei Hauptverzeichnisse:
- **`conf`**: Konfiguration und Assembler‑Startup‑Code
- **`ken`**: Kernel‑Core (Memory Management, Scheduling, …), benannt nach **[Ken Thompson](https://de.wikipedia.org/wiki/Ken_Thompson)**
- **`dmr`**: Treiber & I/O‑Logik, benannt nach **[Dennis Ritchie](https://de.wikipedia.org/wiki/Dennis_Ritchie)**

Da diese Version berühmt dafür ist, **das erste vollständig in C geschriebene UNIX** zu sein, lohnt sich ein Blick in diese Quellen. :-)

### 2. Workspace vorbereiten

Vor dem Start räumen wir alte Object‑Files und Build‑Reste auf, um Platz auf der 2,5‑MB‑Disk zu sparen:

```plaintext
# rm -f *.o
# rm -f lib1 lib2
```

### 3. Core‑Library bauen (`lib1`)

Zuerst wechseln wir in das Verzeichnis `ken`, um die Basis des Betriebssystems zu kompilieren.

1. **Verzeichnis wechseln**:
    ```plaintext
      # chdir ken
    ```
2. **Build‑Script erstellen**: Da es kein `make` gibt, erstellen wir ein Shell‑Skript `mklib.sh` mit dem Editor `ed`. Dieses Skript fügt die Objektdateien nacheinander in das Library‑Archiv ein.
   
   Starte `ed`:
    ```plaintext
      # ed mklib.sh
    ```
   Gib anschließend die folgenden Befehle exakt ein. Tippe `a`, um Text anzuhängen, dann die `ar`‑Zeilen, und beende mit einem Punkt `.` in einer neuen Zeile:
    ```text
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
   _Hinweis: Nach `w` zeigt `ed` die Anzahl der geschriebenen Bytes an._

3. **Kernel‑Sources kompilieren**:
    ```plaintext
      # cc -c *.c
    ```
4. **Archiv‑Skript ausführen**:
    ```plaintext
      # sh mklib.sh
    ```
#### 💡 Hinweis zur Skript‑Ausführung

Du wirst sehen, dass das Skript mit `sh mklib.sh` ausgeführt wird, ohne die Datei ausführbar zu machen. In UNIX V4 war das „executable bit“ für Skripte noch keine übliche Konvention. Indem wir die Datei als Argument an `sh` übergeben, sagen wir der Thompson Shell explizit, die enthaltenen Befehle auszuführen – unabhängig von den Dateirechten.

5. **Cleanup**: Lösche die Objektdateien sofort, nachdem sie in die Library aufgenommen wurden:
    ```text
      # rm -f *.o
    ```
    
### 4. Treiber‑Library bauen (`lib2`)

Jetzt wiederholen wir den Prozess im Verzeichnis `dmr` mit den Quelltexten der Device Driver. Diese Module kümmern sich um die Kommunikation mit Hardware (Disk‑Drives, Konsole, …).

1. **Verzeichnis wechseln**:
    ```plaintext
      # chdir ../dmr
    ```
2. **Treiber kompilieren**:
    ```plaintext
      # cc -c *.c
    ```
3. **In `lib2` archivieren**: Du kannst wie oben schon auch ein Skript `mklib.sh` erstellen, das z. B. folgende Dateien einfügt:
    ```plaintext
      ar r ../lib2 bio.o
      ar r ../lib2 tty.o
      ar r ../lib2 malloc.o
      ar r ../lib2 pipe.o
      ar r ../lib2 cat.o
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
   ... oder schneller noch: Wenn du *alle* kompilierten Treiber aufnehmen willst, kannst du den Archiver auch direkt mit Wildcard verwenden:
    ```plaintext
      # ar r ../lib2 *.o
    ```
4. **Cleanup**:
    ```plaintext
      # rm -f *.o
    ```
    
### 5. Finale Assembly & Konfiguration (`conf`)

Das Verzeichnis `conf` enthält den „Kleber“, der Kernel und Hardware verbindet – hier liegen Startup‑Code (Assembler) und die Konfigurationstabelle.

1. **Verzeichnis wechseln**:
    ```plaintext
      # chdir ../conf
    ```
2. **Low‑Level‑Code assemblieren**: Wir verwenden `as` für Trap‑Vectors und Startup‑Code. Da die Ausgabe standardmäßig `a.out` heißt, müssen wir nach dem Kompilieren die erzeugten Binaries umbenennen:
    ```plaintext
      # as low.s
      # mv a.out low.o
      # as mch.s
      # mv a.out mch.o
    ```
3. **Konfigurationstabelle kompilieren**: `conf.c` definiert, welche Treiber aus der `lib2` tatsächlich in den Kernel gelinkt werden:
    ```plaintext
      # cc -c conf.c
    ```
4. Den **neuen Kernel linken**: Jetzt haben wir `low.o`, `mch.o`, `conf.o` sowie `lib1` und `lib2`. Mit `ld` linken wir alles zu **einem ausführbaren monolithischen Kernel**.

    ```text
      # ld -a low.o mch.o conf.o ../lib1 ../lib2
    ```
   _Hinweis: `-a` erzeugt ein absolutes Executable. Die resultierende Datei heißt `a.out`._

5. **Kernel installieren**: Kopiere oder verschiebe den Kernel ins Root‑Verzeichnis und gib ihm einen passenden Namen:
    ```text
      # cp a.out /unixv4
    ```
6. **Sicher herunterfahren**: Vor dem Testen sicherstellen, dass alle Daten auf das Disk‑Image geschrieben wurden:
    ```text
      # sync
      # sync
      # sync
    ```
---

## Schritt 7: Neuen Kernel testen

Um deinen frisch kompilierten Kernel zu booten, starte den Simulator neu. Wenn der Prompt erscheint, gib statt `unix` deinen neuen Dateinamen ein:
```text
    k
    unixv4
```
Wenn du `mem = ...` und den `login`‑Prompt siehst, hast du UNIX Fourth Edition erfolgreich aus dem Quellcode kompiliert! Es ist Juni 1974 und du arbeitest mit dem ersten C‑basierten Betriebssystem: **UNIX V4**. :-)

* * *

> **Hinweis**: Wenn du diese Skripte oder Anweisungen hilfreich findest, zitiere bitte dieses Repository: [github.com/fhroland/unix_v4](https://github.com/fhroland/unix_v4)

## Credits & Quellen

- **Technische Anleitung**: Basierend auf der hervorragenden Kurzeinführung auf [squoze.net/UNIX/v4/](http://squoze.net/UNIX/v4/)
- **Tape‑Image**: Wiederhergestellt von Thalia Archibald et al. (verfügbar auf [Archive.org](https://archive.org/details/utah_unix_v4_raw))
- **Lizenz**: Dieses Projekt steht unter der BSD 3‑Clause License.

## About

Schritt‑für‑Schritt‑Anleitung, um das wiederhergestellte UNIX V4 „Utah Tape“ (1974, University of Utah) mit SIMH auszuführen.
