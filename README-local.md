# Local notes: AirPlay to the Apple TVs via OwnTone

Set up 2026-11-10 on freshmint (Linux Mint 22.3).

## Why this exists

PipeWire's built-in AirPlay sinks ("Exercise Room", "Living Room (2)", etc. in
Sound Preferences) don't work: every receiver answers `RTSP/1.0 403 Forbidden`
because they require AirPlay 2 pairing, which PipeWire's `raop-sink` can't do.
OwnTone does support AirPlay 2 pairing, so audio goes through it instead.

```
Chrome / any app ──> PipeWire sink "OwnTone (AirPlay 2)"
                 ──> /srv/music/pipewire.fifo
                 ──> OwnTone ──AirPlay 2──> Apple TV(s)
```

## Day to day

- In Sound Preferences, choose **"OwnTone (AirPlay 2)"** for the TVs. Ignore the
  old "Exercise Room" / "Living Room (2)" entries; they will never work.
- Pick rooms and set per-room volume at **http://localhost:3689**. Several rooms
  can play at once.
- The first time you select a new device, it shows a 4-digit PIN on that TV.
  Enter it once in the web page. Exercise Room is already paired.
- Audio lags video by about 2 seconds (normal for AirPlay).

## Switching locations (e.g. the cabin)

Nothing to reconfigure; OwnTone discovers AirPlay devices on whatever network
you're on.

1. Leave Sound Preferences on **"OwnTone (AirPlay 2)"**.
2. Open http://localhost:3689 and click the speaker icon in the bottom bar.
   Turn on the local device(s); turn off any home devices still listed.
3. First time with a pairing device (e.g. an Apple TV): enter the PIN shown on
   its screen. Pairings are remembered, so switching back home needs no PIN.

If a device doesn't appear:

- Make sure the laptop and the device are on the same Wi-Fi (guest networks
  and split 2.4/5 GHz networks often can't see each other).
- `sudo systemctl restart owntone` to re-run discovery.
- Older AirPlay 1 speakers may also work directly from Sound Preferences.

## If sound stops

- Check that the output is still "OwnTone (AirPlay 2)".
- `systemctl status owntone` and `/var/log/owntone.log`
- `sudo systemctl restart owntone`

## Updating / uninstalling

OwnTone is built from this directory and does not auto-update. This is a fork
(`origin` = wytten/owntone-server, `upstream` = owntone/owntone-server); this
file is the only local change. The tree was moved here from `~/src`, so re-run
`configure` before building:

```
cd ~/projects/github/src/owntone-server
git pull upstream master
git push
./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var --enable-install-user
make
sudo make install
```

To uninstall, run the same `./configure`, then `sudo make uninstall`.

## Pieces of the setup

| What | Where |
|---|---|
| OwnTone service | `owntone.service` (systemd, starts at boot) |
| OwnTone config | `/etc/owntone.conf` (defaults; library is `/srv/music`) |
| Audio pipe | `/srv/music/pipewire.fifo` (`wytten:owntone`, mode 0640) |
| PipeWire sink | `~/.config/pipewire/pipewire.conf.d/owntone-sink.conf` |
| Log | `/var/log/owntone.log` |
| Build deps | installed with apt (see OwnTone's "Building" docs) |
