# Android-Linux-Desktop-Setup

Android Linux Desktop

Run a complete Linux desktop environment on an Android phone and access it from another device through a web browser.

The initial MVP uses:

- Android
- Termux
- PRoot-Distro
- Debian
- XFCE
- TigerVNC
- noVNC
- Wi-Fi Hotspot

The goal is to turn an Android phone into a small, portable Linux desktop server that can be accessed from another device connected to the phone's hotspot.

---

MVP Goal

The first milestone is intentionally simple:

Android Phone
     │
     ▼
   Termux
     │
     ▼
 PRoot-Distro
     │
     ▼
   Debian
     │
     ▼
    XFCE
     │
     ▼
  TigerVNC
     │
     ▼
   noVNC
     │
     ▼
 Web Browser

A PC connected to the Android phone's Wi-Fi hotspot should be able to open:

http://PHONE-IP:6080

and interact with the Linux XFCE desktop running on the Android phone.

---

Architecture

                         Android Phone
┌──────────────────────────────────────────────────────┐
│                                                      │
│  Android Wi-Fi Hotspot                               │
│           │                                          │
│           │                                          │
│  ┌────────▼──────────────────────────────────────┐  │
│  │                  Termux                       │  │
│  │                                               │  │
│  │             PRoot-Distro                     │  │
│  │                  │                            │  │
│  │                  ▼                            │  │
│  │               Debian                         │  │
│  │                  │                            │  │
│  │                  ▼                            │  │
│  │                XFCE                          │  │
│  │                  │                            │  │
│  │                  ▼                            │  │
│  │             TigerVNC :5901                   │  │
│  │                  │                            │  │
│  │                  ▼                            │  │
│  │              noVNC :6080                     │  │
│  │                                               │  │
│  └───────────────────┬───────────────────────────┘  │
│                      │                              │
└──────────────────────┼──────────────────────────────┘
                       │
                 Wi-Fi Hotspot
                       │
                       ▼
                    PC/Laptop
                       │
                    Browser
                       │
                       ▼
             http://PHONE-IP:6080

---

Components

Termux

Android terminal and Linux environment.

PRoot-Distro

Provides rootless Linux environments inside Termux.

It allows us to run a Linux userspace without replacing Android's kernel.

Debian

The Linux distribution used for the initial MVP.

XFCE

Lightweight Linux graphical desktop environment.

TigerVNC

VNC server that creates the graphical desktop display.

noVNC

Browser-based VNC client.

WebSocket

Persistent browser-to-server communication channel used by noVNC.

websockify

Translates WebSocket traffic into the TCP traffic expected by the VNC server.

---

Installation

1. Install Termux

Install Termux from one of the official sources:

- F-Droid
- Termux GitHub Releases

Do not mix Termux APKs or plugins from different signing sources.

---

2. Update Termux

Open Termux:

pkg update
pkg upgrade -y

Install basic tools:

pkg install -y git curl wget nano proot-distro iproute2

---

3. Install Debian

Install PRoot-Distro:

pkg install -y proot-distro

Check available distributions:

proot-distro list

Install Debian:

proot-distro install debian

Enter Debian:

proot-distro login debian

You should now see a Linux shell.

Check:

cat /etc/os-release

Check architecture:

uname -m

---

4. Update Debian

Inside Debian:

apt update
apt upgrade -y

---

5. Install XFCE

Install the desktop environment:

apt install -y xfce4 xfce4-goodies dbus-x11

---

6. Install TigerVNC

apt install -y tigervnc-standalone-server tigervnc-tools

Check:

vncserver --version

---

7. Configure the VNC desktop

Create the VNC configuration directory:

mkdir -p ~/.vnc

Create the startup script:

nano ~/.vnc/xstartup

Add:

#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS

export XDG_CURRENT_DESKTOP=XFCE
export XDG_SESSION_DESKTOP=xfce
export DESKTOP_SESSION=xfce

dbus-launch --exit-with-session startxfce4

Save the file.

Make it executable:

chmod +x ~/.vnc/xstartup

---

8. Create the VNC password

Run:

vncpasswd

Create a password when prompted.

Do not use a password that you use for important accounts.

---

9. Start the Linux desktop

Start display ":1":

vncserver :1 -geometry 1280x720 -depth 24

Display ":1" normally corresponds to TCP port:

5901

Check:

ss -lntp | grep 5901

Expected:

LISTEN ... 5901

At this point the Linux desktop is running.

---

10. Test the VNC server locally

Inside Debian:

vncserver -list

You should see something similar to:

TigerVNC server sessions:

X DISPLAY #     PROCESS ID
:1              XXXXX

The important value is:

:1

which corresponds to:

localhost:5901

---

11. Install noVNC

Inside Debian:

apt install -y novnc websockify

Check:

which novnc_proxy

and:

which websockify

---

12. Start noVNC

Start the browser gateway:

novnc_proxy --vnc localhost:5901 --listen 0.0.0.0:6080

The important parameters are:

--vnc localhost:5901

Connect noVNC to the VNC server.

And:

--listen 0.0.0.0:6080

Listen for browser connections on port 6080.

---

13. Connect the PC to the Android hotspot

Enable the Android Wi-Fi hotspot.

Connect the PC/laptop to it.

Find the phone's hotspot IP address from Termux:

ip -4 addr

You can also inspect:

ip route

Look for the interface/IP used by the hotspot.

The address may look like:

192.168.43.1

or:

192.168.1.1

or another private address depending on the Android device.

---

14. Open the Linux desktop from the PC

On the PC connected to the phone hotspot, open:

http://PHONE-IP:6080

For example:

http://192.168.43.1:6080

The noVNC interface should appear.

Click:

Connect

Enter the VNC password.

You should now see the XFCE Linux desktop.

---

Troubleshooting

Check Debian

cat /etc/os-release

Check XFCE

which startxfce4

Check VNC

vncserver -list

Check port 5901

ss -lntp | grep 5901

Check noVNC

ss -lntp | grep 6080

Check processes

ps aux | grep -E 'vnc|xfce|novnc|websockify'

Stop VNC

vncserver -kill :1

Restart VNC

vncserver -kill :1
vncserver :1 -geometry 1280x720 -depth 24

Then restart noVNC:

novnc_proxy --vnc localhost:5901 --listen 0.0.0.0:6080

---

Important Security Note

The MVP intentionally focuses on functionality.

Do not expose port "6080" to the public Internet.

The intended network is:

PC
 │
 └── Android Wi-Fi Hotspot
          │
          └── Linux Desktop

Future versions should add:

- Authentication
- HTTPS/WSS
- Session management
- Access control
- Better network binding
- Automatic startup/shutdown
- Web dashboard

---

Project Roadmap

MVP-1

- [x] Android device
- [x] Termux
- [x] PRoot-Distro
- [x] Debian
- [x] XFCE
- [x] TigerVNC
- [x] noVNC
- [ ] PC browser access
- [ ] End-to-end desktop session

MVP-2

- [ ] One-command installer
- [ ] Start/stop scripts
- [ ] Status command
- [ ] Automatic IP detection
- [ ] Logging
- [ ] Configuration file

MVP-3

- [ ] Web dashboard
- [ ] Authentication
- [ ] Session management
- [ ] Automatic desktop startup
- [ ] Resolution selection

Future

- [ ] Multiple distributions
- [ ] Container management
- [ ] Persistent storage
- [ ] Application management
- [ ] File browser
- [ ] Terminal web interface
- [ ] HTTPS/WSS
- [ ] Remote access
- [ ] Resource monitoring

---

Current Goal

The only goal for the first milestone is:

Android Phone
      │
      ▼
    Termux
      │
      ▼
   Debian
      │
      ▼
    XFCE
      │
      ▼
   TigerVNC
      │
      ▼
    noVNC
      │
      ▼
 Android Hotspot
      │
      ▼
      PC
      │
      ▼
    Browser
      │
      ▼
 Linux Desktop

If this works, MVP-1 is complete.

---

License

To be decided.
