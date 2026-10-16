# RE / security roundup (posted 2026-10-15)

| # | Post | Live | Image |
|---|---|---|---|
| 1 | PentAGI v2.2.0 | https://x.com/P7RIM3/status/2110738836565233760 | 01-pentagi-v2.2.0.png |
| 2 | radare2 6.2.4 | https://x.com/P7RIM3/status/2110742726958276756 | 02-radare2-6.2.4.png |
| 3 | Frida 17.23.3 | https://x.com/P7RIM3/status/2110746639119892577 | 03-frida-17.23.3.png |
| 4 | TP-Link EAP770 kernel exploit (thread) | https://x.com/P7RIM3/status/2110750663000904003 | 04-eap770.png |
| 5 | Insta360 Wave open firmware | https://x.com/P7RIM3/status/2110754427304845642 | 05-insta360-wave.png |
| 6 | SneakPeek (thread) | https://x.com/P7RIM3/status/2110758262459253062 | 06-sneakpeek.png |
| 7 | Xbox 360 glitching | https://x.com/P7RIM3/status/2110762136460726557 | 07-xbox360-hn.png |

---

## 1. PentAGI v2.2.0
PentAGI 2.2 closes CVE-2026-14784. Up to 2.1.0, the shipped .env.example set DOCKER_INSIDE=true, which mounted the host Docker socket into every sandbox. A prompt-injected agent could take over the host.

The fix drops every container capability and holds back MKNOD (the main known escape path). DOCKER_INSIDE now defaults to false. Optional startup tests check that 27 host-escape requests get refused.

The team says it plainly: prompt injection can't be fully closed, so this release is about containing whatever runs in the sandbox. That's the right way to think about AI pentest agents. If you're running PentAGI with an old .env, how sure are you that socket isn't still mounted?

https://github.com/vxcontrol/pentagi/releases/tag/v2.2.0

## 2. radare2 6.2.4
radare2 6.2.4 "The Magnetician" is out: 453 commits from 18 contributors since 6.2.2.

The part I'd care about: a long list of sandbox hardening. Shell subcommands are blocked in sandbox mode, a sandbox escape through an e- config reset is fixed, and a shell injection in curl headers is closed. There are also iOS 27 dyldcache support and better ARM64 switch recovery.

Your RE tool parses hostile files all day, so it needs hardening too. When did you last update yours?

https://github.com/radareorg/radare2/releases/tag/6.2.4

## 3. Frida 17.23.3
Frida 17.23.3 brings the .NET bindings back. @AeonLucid rewrote them from scratch as plain C# on .NET 9. They're generated from the same GIR as the Python, Node and Swift bindings, so they cover the full API and run on Windows, macOS and Linux.

Heads up: the API isn't stable yet and it's not on NuGet, so for now you build it from the repo. The release also fixes a code-slice leak on RWX-capable hosts.

Anyone in the C# crowd finally going to try Frida?

https://frida.re/news/

## 4. TP-Link EAP770 kernel exploit (4-post thread)
1/ Guy's TP-Link EAP770 was corrupting big Wi-Fi downloads. The fix was writing two zeros to sysfs, but TP-Link's shell won't let you write as root and the device was EOL with no updates. So he wrote a kernel exploit for his own access point. Receipts 🧵

2/ The bug: Qualcomm's NSS driver reuses packet buffers without fully syncing the CPU cache. Stale 64-byte cache lines overwrite received packets, then the offload engine recalculates a valid TCP checksum over the garbage. TCP accepts it, and only TLS and SSH MACs catch it ("bad record mac").

3/ The way in was CVE-2022-0185 on a Linux 5.4.164 kernel with no KASLR, plus pipe-buffer tricks to overwrite modprobe_path and get root. He built it with Opus 4.8 in about 17 hours, tested it in QEMU first, and gets one shot per boot. Turning off the offload cost about 8% speed and dropped bad blocks to zero.

4/ Unsupported hardware forced the owner to exploit his own device just to fix it. How many EOL boxes are sitting in your network with the same problem?
https://blog.dfb.sh/blog/eap770-root-for-two-zeros/

## 5. Insta360 Wave open firmware
Someone wrote open firmware for the Insta360 Wave speakerphone. Here's what their writeup says the stock firmware does: sends 35+ kinds of usage events to ByteDance's DataFinder tagged with the serial number, routes meeting audio, transcripts and voiceprints to Insta360's cloud, and signs updates with a 512-bit RSA key. Older releases (up to 1.0.13.6) opened a root telnet shell with the password 123456.

waved replaces it with one Rust program that keeps recordings on the device, and one script reverts to stock. It's unofficial and not affiliated with Insta360.

That's a meeting-room mic. Do you know where your conference audio goes?

https://xaviergeerinck.github.io/insta360-custom-firmware/

## 6. SneakPeek (4-post thread)
1/ Noma Labs found that Claude Code Desktop's preview pane could leak your secrets just by rendering a file. Clone a repo, ask Claude to look at the "coverage report," and SSH keys, cloud creds and tokens are gone. Anthropic scored it CVSS 8.7. Receipts 🧵

2/ How it worked: files over 256KB load as file:// URLs instead of data: URLs, and that path skipped the CSP. The request filter only ran when proxying a dev server, not on static files. So padded HTML ran JavaScript, fetch()'d local files and sent them out with sendBeacon. No permission prompt.

3/ Anthropic's first patch blocked fetch() to file:// paths. Noma bypassed it with a blob: Web Worker. Anthropic shipped the full fix in Claude Desktop 1.30096.1+ (Aug 17) via auto-update. The CVE is still pending.

4/ Your agent opening untrusted repos is now an attack surface. Make sure you're updated. Do you review repos before letting your agent touch them?
https://noma.security/noma-labs/sneakpeek-how-the-preview-pane-in-claude-code-desktop-can-exfiltrate-your-data

## 7. Xbox 360 glitching
Bought a Matrix CoolRunner glitcher in 2017, gave up, and finished the Xbox 360 mod in 2026 with a £5 Raspberry Pi Pico. That's the story in "It took me nine years to hack my Xbox 360."

The trick is the reset glitch: pulse CPU_RESET for about 100ns at the right moment and an instruction in signature verification moves a 0, so code runs unsigned. A NAND dump that would've taken 30+ minutes over an LPT port took seconds with PicoFlasher.

The bigger point he makes: as software security hardens, fault injection gets more attractive and cheaper. Think ESP32 secure boot bypass, AirTag dumps, PicoFly on the Switch. Is a hardware glitch the new default attack path?

https://kennedn.com/blog/posts/glitching/
(Image is the HN thread title; the article page wouldn't render for a screenshot.)
