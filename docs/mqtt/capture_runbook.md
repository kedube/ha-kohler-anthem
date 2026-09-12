# Capturing diagnostics

How to gather evidence for a bug report, or to see exactly what Kohler's cloud is sending.
Three tools cover this, from quickest to most detailed.

## Download diagnostics

Every device page and the integration card have a **Download diagnostics** button. It
produces one JSON report describing the whole installation — model and outlet split as
detected, what each device is reporting, configured limits — with credentials, account
identity, and device serial numbers redacted.

This is the single most useful thing to attach to an issue, especially on hardware other
than a K-28212.

## The Report Log switch

The quickest way to capture live MQTT traffic for a bug report. A **Report Log** switch sits
on both device pages, in the diagnostic section:

* **Switch on** → a new capture file starts, recording every raw MQTT message as received,
  before any decoding.
* **Restart Home Assistant mid-capture** → the same file continues, so a restart never
  splits the evidence.
* **Switch off** → the capture ends. The next switch-on starts a fresh file.

Files land in `custom_components/kohler_anthem/reports/` (a `README.txt` there explains the
format), one per capture, capped at 8 MB with continuation parts.

**Check the file before sharing it** — it contains your device identifiers and shows when
the shower was used. The folder lives inside the integration itself, so **updating or
reinstalling the integration deletes it**; move files you want to keep first.

## The two debug loggers

For a lower-level view than the diagnostics report or the Report Log — every byte Kohler's
cloud actually sends, including fields the integration doesn't read — two debug loggers
capture the two halves of the traffic. Both are off by default, neither survives a restart,
and both are switched on from **Developer Tools → Actions**, `logger.set_level`, in YAML
mode:

```yaml
action: logger.set_level
data:
  # Every REST call: endpoint, status, and the full response body.
  custom_components.kohler_anthem.anthem.client: debug
  # Every MQTT payload, exactly as it arrived and before any decoding.
  custom_components.kohler_anthem.anthem.raw_log: debug
```

Set either back to `info` to stop. Read the results in **Settings → System → Logs**, or in
`home-assistant.log`.

Use these to answer questions the diagnostics report can't — whether a field exists at all,
what an undocumented value looks like on your hardware, or what an endpoint returns on a
system unlike the reference install.

**Credentials are redacted** from the `client` log before it is written: the mobile-settings
call returns a short-lived IoT Hub password, and any key or value that looks like a
password, token, key or secret is replaced.

Everything else is logged in full — including device ids and serial numbers, which the
diagnostics report redacts but a raw debug log does not. **Skim a log before attaching it to
an issue.** A Kohler device id is not merely an identifier: it is the address the cloud uses
to reach your valve.

The ordinary `home-assistant.log` stays free of device ids at INFO level and above; turning
either debug logger on puts ids and serials in that file deliberately, and they stay there
until it rotates.
