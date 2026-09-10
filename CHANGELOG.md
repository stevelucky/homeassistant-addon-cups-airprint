## v1.9

- Pin Avahi to a single network interface. `enable-reflector=no` in v1.8 reduced
  the self-collision but did not remove it: with `host_network: true` Avahi still
  *binds* wlan0, hassio, docker0, tailscale0 and every veth pair, hears its own
  announcement arrive on a second interface, and renames itself. A live install on
  v1.8 was still coming up as `9e3ebd7e-cupsik-3.local`.

  The damage is that cupsd's service record keeps advertising the ORIGINAL host
  name, which no longer resolves:

      ping 9e3ebd7e-cupsik.local    -> Unknown host        (what SRV points at)
      ping 9e3ebd7e-cupsik-3.local  -> replies             (what Avahi registered)

  Everything that follows the advertisement therefore fails after a successful
  discovery. iOS lists the printer, spins on "Gathering printer information", then
  reports "The printer is offline". macOS cannot fetch capabilities and offers only
  Generic PostScript instead of AirPrint. Printing by IP works the whole time,
  which is what makes it look like a driver problem rather than a name problem.
  The TXT record itself is fine - `URF`, `pdl` and `rp` are all correct.

  The avahi-daemon run script now writes `allow-interfaces=` before starting the
  daemon, taking the value from the new `mdns_interface` option, or auto-detecting
  the host's default-route interface from /proc/net/route when that is left empty.

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
