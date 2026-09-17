## Rafiul Bari

Cybersecurity student at NYC College of Technology. I build measurement tools, and
I'm mostly interested in the question underneath them: **how do you know your
check actually checked anything?**

That sounds like a philosophy problem. In practice it's the most common bug I
find. A scanner returns `clean` because it never ran. A sandbox reports `blocked`
because the probe was unprivileged. A build diff says `unchanged` because both
builds failed identically. Each of those is a passing result produced by an
absence, and none of them look wrong from the outside.

So the repos below all try to do the same thing: **state what they cannot
establish, as prominently as what they can.** Every README documents its own
defects, with the measurement that found each one.

---

### 🔍 [aur-build-diff](https://github.com/blurxy/aur-build-diff)
*Build an AUR package twice — this version and the last known good one — in an isolated sandbox, and diff what each one actually does.*

AUR packages are arbitrary shell scripts that run on your machine. This builds two
revisions under `bubblewrap` with `strace` and reports what the newer one does that
the older never did: new outbound endpoints, binaries it didn't execute before,
writes outside the build tree.

Two things worth reading it for:

- **The sandbox blocks DNS through the `io.systemd.Resolve` unix socket, not just
  the network namespace.** `--unshare-net` alone still resolves names — that's a
  control the repo runs on every check, and it's why the naive version reports
  `dns:RESOLVED` next to the hardened one's `dns:blocked`.
- **`--static` reads what a PKGBUILD *declares*, in 1.3s with no sandbox** — and
  it's the *only* thing that catches a package which never fetched and suddenly
  does, because proving that behaviourally needs a completed build, and completing
  the build is what the sandbox exists to prevent. It fires on exactly one of six
  real `pkgcacheclean` transitions and stays silent on the other five.

The README documents **eight defects found in the tool**, plus three more in the
`--static` differ itself — those three were in code I'd written an hour earlier,
and seven of my own passing tests caught none of them.

### 📡 [beacon-spectrum](https://github.com/blurxy/beacon-spectrum)
*Finding periodic C2 beacons in network timing, with a Rayleigh/Z² periodogram.*

Detects beaconing from connection timestamps alone — no payload, no TLS
inspection. Dependency-free pcap and Zeek ingest.

The part I'd point at is the table it publishes **against itself**. A
dispersion/skew control detector wins every headline benchmark — ~100% detection,
including 600s beacons at 70% jitter. It also alarms on **98% of benign periodic
traffic**: NTP, telemetry, backups. It isn't detecting beacons, it's detecting
regularity.

It also derives its own hard limit rather than discovering it empirically:
`Z² = 2N·R²` and `R ≤ 1`, so the statistic is **bounded by 2N** — a perfectly
periodic beacon observed N times cannot score above twice its check-in count. Below
that floor the honest output is *"not enough data"*, not *"no beacon"*.

### 🎼 [chladni-tui](https://github.com/blurxy/chladni-tui)
*A Chladni sand plate simulated from first principles and drawn in the terminal.*

Bessel eigenmodes of a clamped circular membrane, `U_mn(r,θ) = J_m(α_mn·r)·cos(mθ)`,
driven by live audio and rendered in Braille sub-cells.

The physics note I like best: the energy is an **incoherent** sum `Σaᵢ²Uᵢ²`, not
`(ΣaᵢUᵢ)²`. Squaring a coherent sum invents interference nodes no real plate has.
The consequence is visible — one clean tone gives you lines, a chord gives you
isolated *dots*, because the common zeros of several independent functions are
points, not curves.

It also documents where it departs from a real Chladni plate: a metal plate has a
*free* edge and obeys the biharmonic equation, so its modes aren't Bessel functions
at all. This is the clamped-membrane model, and says so.

---

### Also public

[**spore**](https://github.com/blurxy/spore) — an off-web mycelial mesh; runs on one
device and gets faster as you add more ·
[**py-exec-mcp**](https://github.com/blurxy/py-exec-mcp) — run Python from an MCP
client without a shell mangling your quoting ·
[**project-x-py**](https://github.com/blurxy/project-x-py) — async Python SDK for the
ProjectX trading platform

---

### The recurring bug, if you only read one thing

Across all of this, the same defect keeps appearing in different costumes: **a check
that is correct about what it examined, and silent about what was actually asked.**

- `nmap -sn` unprivileged finds 2 hosts; as root, 4. Same command, same network.
- A TCP probe says the sandbox has no network. DNS still resolves — over a unix socket.
- A build diff says `unchanged`. Both builds failed identically.
- `pgrep -f pattern` returns a hit. The hit is its own command line.
- Seven unit tests pass. All seven encode the same wrong assumption the code does.

The last one is the one I'd underline. A fixture you wrote can't fail where your
model is wrong, because the fixture and the bug have the same parent — which is why
the fix isn't *more tests*, it's running the thing against real data you didn't
author. One pass over six real package revisions found two defects that seven
deliberate tests missed.

📫 Reachable through GitHub.
