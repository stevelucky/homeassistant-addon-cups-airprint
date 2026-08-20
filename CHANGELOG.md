## v1.8

- Ship the Avahi reflector fix. `enable-reflector=no` was committed in July but
  never reached any installation, because the add-on version was left at 1.7 and
  Home Assistant only offers an update when the version string changes.

  With the reflector on and `host_network: true`, Avahi re-broadcasts its own
  announcements across every interface the host has — wlan0, hassio, docker0 and
  a dozen veth pairs. It then hears itself, treats the echo as another machine
  claiming the name, and renames itself. Observed in the wild flipping between
  `9e3ebd7e-cupsik-2` and `9e3ebd7e-cupsik-3`, ten conflicts deep.

  Symptom this fixes: the printer is discoverable from iOS/macOS, but printing
  fails with a "contacting printer" error. Discovery succeeds against a cached
  service record; by the time a job is submitted the advertised `.local` hostname
  has been withdrawn and no longer resolves. Printing from the CUPS admin page
  keeps working throughout, because that path uses the IP and never touches mDNS.

## v1.7

- Fix the issue with "ulimit" size and permissions
  
## v1.5

- Update Debian Base to 7.6.2
- Add HP drivers (printer-driver-hpcups)
