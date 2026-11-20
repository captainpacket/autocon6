# Questions from the talk

Questions that came in after AutoCon 6, with the answers as best I can give them. Opinions are mine, and where I don't have a measurement I say so.

## Did the Docker-in-Docker cost show up in boot time or in the data plane?

Mostly boot time, and then not being able to see what was going on. That's from my experience with clabernetes, which runs containerlab inside Kubernetes. I never ran it head to head against KNE on the same topology, so treat this as experience, not a benchmark.

- **Boot time.** Labs took a long time to come up, and a lab that's slow to boot is a lab people stop rebuilding. For a disposable lab platform that matters a lot.
- **Opaque observability.** With Docker running inside pods, it was hard to tell what state a lab was in. Was the device up? Was the link up? Where did it stall? That's the same problem as the checks in the talk. If you can't see inside, "it booted" tells you very little.

I can't give you a data plane number. That wasn't what pushed me off it.

**Update:** a clabernetes maintainer got in touch to say the nested setup is gone. The release notes agree: v0.9.0 (30 Sep 2026) replaced the nested launcher runtime with a native Kubernetes runtime, and the architecture docs now say each node is a device Pod with no Docker daemon inside it. My experience above is from before that, so it describes the old design. I haven't tried 0.9, so I can't tell you whether boot time or observability got better. Worth another look. Details at [c9s.run](https://c9s.run).

I'm not rushing back, for one reason. For VM-based devices, 0.9 still runs the VM inside a container through the vrnetlab entrypoint, and that hides what the VM is doing. Running them on KubeVirt natively makes the VM visible to Kubernetes, which is what I wanted. That only affects VMs. If your devices are container-native, like SR Linux, the 0.9 change may be all you need.

There's also a fit problem. containerlab wants a container per device, which is fine for container-native NOSes (SR Linux is one). For devices that ship as VMs you end up with a VM inside a container inside a pod, and each layer has its own opinions about networking and resources. [Slide 7](SLIDES.md#7-why-not-just-use-containerlab) is the short version.

KNE gives me a pod or a VM per device, and Kubernetes places each one. That isn't free either. I carry a meshnet fork, the MTU between nodes is lower, and `Eth0/2` isn't a legal Linux interface name, so the pod waits forever. I'd still make the same call, but it moved the cost around, it didn't remove it.

If your devices are all container-native you may never feel most of this. What I'd measure if I were comparing them: time from "create" to first ping across a real link, and then a traffic generator on the data plane, not just ping. I have the second half (Open Traffic Generator, proven on two labs, layer 2 only) but not the comparison.

## How do you prove that each tenant's lab actually works?

This is the part of the talk I'd copy first, so it's worth saying plainly.

- **Test across a real link, not the management network.** A green boot is not a working network. In the talk the management network was all green while the data link was dead ([slide 18](SLIDES.md#18-a-green-boot-is-not-a-working-network)).
- **Make every check fail once.** If you've never seen a check fail, you don't know it can. I break the thing on purpose and see whether the check notices.
- **Don't trust a fake that can't fail.** My fake device passed tests that the real firewall refused ([slide 23](SLIDES.md#23-the-fake-passed-the-firewall-didnt)).
- **Hold every change until something that isn't you has judged it.**

[Slide 31](SLIDES.md#31-copy-this-skip-that) has the full copy-this and skip-that list.

One caveat: this is one lab platform for SEs and my own testing. A multi-tenant training platform with a portal, an IDE, AWX and NetBox has more moving parts to validate than I've had to, and I'd like to hear how you structured yours.
