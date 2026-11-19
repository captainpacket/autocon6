# Every Automation Demo Lies

Slides and context for my talk at [AutoCon 6](https://networkautomation.forum) (Network Automation Forum), 19 November 2026.

Craig Johnson, Forward Networks. This is my personal repo, so the opinions are mine.

## What the talk is about

I build a lab platform called Skyforge. It started because SEs kept needing a lab they could break and get back: disposable, shareable, safe to learn on, the same tomorrow as it was today.

That's the demo story. It's also a network automation story. A lab you can build from a declarative description, push a change into, and throw away is a repeatable place to test network changes before they touch production. The demo environment and the change-testing environment turned out to be the same thing.

The talk is mostly about what went wrong on the way. The short version:

> If you want reliable automation, you need somewhere reliable to break it first.

## The slides

| | |
|---|---|
| [`slides/AC6_Every_Automation_Demo_Lies.pdf`](slides/AC6_Every_Automation_Demo_Lies.pdf) | PDF. Every click-reveal is shown at once. |
| [`slides/AC6_Every_Automation_Demo_Lies.pptx`](slides/AC6_Every_Automation_Demo_Lies.pptx) | PowerPoint, with the click-by-click reveals. Speaker notes removed. |
| [`SLIDES.md`](SLIDES.md) | Every slide as an image, with titles, so you can read it in the browser. |

33 slides in the talk, plus 9 backup slides at the end (extra detail, a status table, and the act dividers).

## The story

**Why I built it (slides 1 to 5).** A demo that works for one person on one lab isn't a demo of anything. Everyone at once is the test. I open with a spoiler: a real bad change, on a live lab, that undid itself in about 30 seconds.

**How I built it (slides 6 to 19).** The stack, and the parts I got wrong. The platform was wrong twice (containerlab and clabernetes, then k3s on hosts I built), and I ended up on KNE running on Talos Kubernetes. Then the argument for declarative: netlab accepted a topology, compiled it and booted it, and 77 of 128 probes lost everything. One line (`ospf: false`) fixed it, and the next run passed 264 of 264. A compile is not a boot, and a green boot is not a working network.

**How we use it (slides 20 to 28).** Every change is staged, predicted, approved, pushed, judged, then kept or undone. The judge is the point. A push library told me "reverted" when the device had lost about half its config. A fake device passed tests that a real firewall refused. So the pattern is: hold every change until something that isn't me judges it, and if nobody confirms, it reverts. Slide 29 is a scorecard of which platforms do that today and how each one reverts.

**Close (slides 29 to 33).** Where it stands, then the ask, then what to copy and what to skip.

## The ask

Every vendor has its own way to say "add this route". netlab gets the network close to right. Infrahub describes what should exist, but it isn't the change. There's no shared, typed, vendor-neutral language for the change itself.

I'd like to make that a Network Automation Forum project. It shouldn't be mine. I've proven the idea on a handful of device families, and I'd like your edge cases. If you've got some, open an issue on this repo or find me at AutoCon.

## What's proven and what isn't

I only put things on slides that I've run. Backup slide 38 is a one-line status for each item, and "Planned" means planned. The things I haven't done yet are on that slide, not hidden.

## Skyforge

[github.com/forwardnetworks/skyforge](https://github.com/forwardnetworks/skyforge) has the patches, the netlab topologies and an install guide, which is what you need to build something similar. The control plane isn't published yet, and I don't have a date.

## What it's built on

- [netlab](https://netlab.tools): turns one topology file into a full config for every device
- [KNE](https://github.com/openconfig/kne) (Kubernetes Network Emulation): a pod or a VM per device
- [meshnet-cni](https://github.com/networkop/meshnet-cni): wires the links between pods (I carry a fork)
- [Talos Linux](https://www.talos.dev): Kubernetes with nothing to log into
- [Flux](https://fluxcd.io): git is what runs
- [Infrahub](https://github.com/opsmill/infrahub): the schema format I use for rules about the network
- [scrapligo](https://github.com/scrapli/scrapligo): device pushes (the PAN-OS push uses its API)
- [Open Traffic Generator](https://otg.dev): real traffic, not just ping
- [containerlab](https://containerlab.dev): the thing everyone asks about. Slide 7 is why I didn't use it
- Forward Networks' digital twin for the predict and verify steps

## Notes

- The slides include memes and screenshots from other people's work (Skyrim, Twilight, The IT Crowd, Spider-Man, xkcd and others). They belong to their owners and are there as commentary.
- No license is set on this repo yet, for that reason.
