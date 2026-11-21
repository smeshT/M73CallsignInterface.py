# M73CallsignInterface.py

A Reticulum interface that puts a station callsign on every frame, for Modem73 over the amateur bands.

```
M73:KJ5FQT:<the original Reticulum frame>
```

On RDM-300, RDM-600 and MFSK, Modem73's encoders do not receive `call_sign` at all, so a stock
station transmits frames with no station identification inside them. M73_RNS closes that gap by
prepending a plain-text callsign. Eleven bytes for a six-character callsign: ASCII, unencrypted,
visible to anyone listening.

The purpose is station identification under 47 CFR §97.119.

## Installation

Copy `M73CallsignInterface.py` into `~/.reticulum/interfaces/`. The filename must be exactly
`M73CallsignInterface.py`, because Reticulum derives the file to load from the `type` value.

This file subclasses the stock `Modem73Interface`. Leave that file where it is; the import fails
without it.

For the operating mode, the registry, and the ratchet requirement, see **M73_RNS**.

## Config

```ini
[[Modem73 Callsign]]
    type = M73CallsignInterface
    enabled = true
    envelope = true
    callsign = YOURCALL
    operator_log = ~/.reticulum/m73log
    target_host = 127.0.0.1
    target_port = 8001
    control_host = 127.0.0.1
    control_port = 8073
    mode = internal
    kiss_framing = true
    recursive_prs = no
    path_discovery = passive
    announces_from_internal = yes
    announces_to_internal = yes
```

- `callsign` — required when `envelope = true`; the interface refuses to load without it.
- `envelope` — `true` wraps frames. `false` behaves like the stock interface.
- `strict` — defaults to `true`. Logs a warning when a frame arrives without an envelope.
- `operator_log` — optional path for the per-frame station log.
- `path_discovery` — `discover` (default) or `passive`.
- `recursive_prs` — stock Reticulum key, inherited from the base interface.

### path_discovery

`recursive_prs = no` only stops path requests that *start* on this interface. It does not stop the one
Transport generates on its behalf: when a local client asks for an unknown destination, Transport
searches on every other online interface, HF included.

`path_discovery = passive` drops those discovery sweeps at the modem boundary. Requests for a
destination already in the local path table still go out, because those are direct answers to a peer.

Leave `recursive_prs = no` in place. The two keys cover opposite directions — one governs inbound,
one governs outbound — and neither replaces the other.

## Log

```
2026-11-19T14:42:55Z   tx         KJ5FQT     194B
2026-11-19T16:53:47Z   rx         KF0YRY/2   183B
2026-11-19T16:54:01Z   rx_noenv   -          241B
```

Timestamp, direction, callsign and byte count only. No message content.

## License

MIT
