+++
title = "AI on Fedora"
date = 2025-02-06
prev = '/docs'
weight = 3
+++

Fedora is an awesome Linux distribution, but what makes it even more awesome is
the fact that you can run AI on it fairly easily. That is, if you know how to
run it first, which is the hard part. This article goes through some technical
details of getting certain Fedora-specific problems out of the way. Since most
Linux guides regarding AI cater to Debian-based or Arch-based systems, I have
found very little information on how to get some AI tools running on Fedora.

{{< figure src="/ai_on_fedora/fedora_gigachad.webp" caption="Tips my AI-generated fedora in your direction." >}}

As a proud Fedora user, I will share the knowledge I gathered. My computer setup:

- **{{< icon "cog" >}} OS**: Fedora 41
- **{{< icon "lightning-bolt" >}} GPU**: Radeon RX 6700 XT
- **{{< icon "chip" >}} CPU**: AMD Ryzen 7 2700X
- **{{< icon "database" >}} RAM**: 32 GB

Using this, I managed to run:

- [LostRuins/koboldcpp/](https://github.com/LostRuins/koboldcpp/) (LLM; text
generation)
- [YellowRoseCx/koboldcpp-rocm](https://github.com/YellowRoseCx/koboldcpp-rocm) (LLM)
- [oobabooga/text-generation-webui](https://github.com/oobabooga/text-generation-webui)
(LLM)
- [comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI)
(Diffusion; image generation)

## Getting the common dependencies

There are some dependencies you will need, regardless of what kind of AI you
are going to use. Thankfully, Fedora does actually have all of them available
in the repositories, but they are not obvious to find. The main source of
information on those dependencies is [this fairly obscure Fedora Wiki
page](https://fedoraproject.org/wiki/SIGs/HC).

### ROCm

ROCm (short for *Radeon Open Compute*) is AMD's poor answer to NVIDIA's CUDA
technology. It's an open-source software platform designed to support
high-performance computing (HPC) and artificial intelligence (AI) workloads on
AMD GPUs. It includes a bunch of drivers, development tools, and APIs, but
those are highly disorganised in comparison to CUDA. 

The main piece of information to know is that the range of hardware supported
by ROCm is extremely narrow. NVIDIA GPUs from 2013 or even 2011 can be used to
do LLM inference. The same can't be said about AMD GPUs. It's only recently
that it started officially supporting RDNA 2.0 cards like the RX 6700 XT.
[Here's a full
list](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html)
of cards that are officially supported by ROCm. It's not an impressive list.
There are many unofficially supported devices. I've read posts on the Internet
about people getting RX 580s to work with ROCm, but don't hold your breath.
ROCm is not even supported on Windows at all, at least in practice. AMD users
on Windows need to use [workarounds like
DirectML](https://github.com/comfyanonymous/ComfyUI?tab=readme-ov-file#directml-amd-cards-on-windows)
instead.

That being said, **you want to use ROCm at every available opportunity, as it
is the fastest backend to use with AMD GPUs for any AI workload.**

Should you have an AMD GPU architecture supported by ROCm, you will need to
install the following packages:

```fish
sudo dnf install rocm-hip-devel rocm-hipblas-devel rocm-llvm-devel rocminfo hipblas-devel hipcc
```

While it is not strictly necessary to install the development (`*-devel`)
versions of these packages, practically all AI tools on Linux will require you
to compile the tools from source if you are using ROCm. Therefore, you might as
well get the full suite.

**To verify ROCm functionality on your system**, run `rocminfo`. You should get
output like this:

{{% details title="Long terminal output" closed="true" %}}

```txt
ROCk module is loaded
=====================
HSA System Attributes
=====================
Runtime Version:         1.1
Runtime Ext Version:     1.6
System Timestamp Freq.:  1000.000000MHz
Sig. Max Wait Duration:  18446744073709551615 (0xFFFFFFFFFFFFFFFF) (timestamp count)
Machine Model:           LARGE
System Endianness:       LITTLE
Mwaitx:                  DISABLED
DMAbuf Support:          YES

==========
HSA Agents
==========
*******
Agent 1
*******
  Name:                    AMD Ryzen 7 2700X Eight-Core Processor
  Uuid:                    CPU-XX
  Marketing Name:          AMD Ryzen 7 2700X Eight-Core Processor
  Vendor Name:             CPU
  Feature:                 None specified
  Profile:                 FULL_PROFILE
  Float Round Mode:        NEAR
  Max Queue Number:        0(0x0)
  Queue Min Size:          0(0x0)
  Queue Max Size:          0(0x0)
  Queue Type:              MULTI
  Node:                    0
  Device Type:             CPU
  Cache Info:
    L1:                      32768(0x8000) KB
  Chip ID:                 0(0x0)
  ASIC Revision:           0(0x0)
  Cacheline Size:          64(0x40)
  Max Clock Freq. (MHz):   3700
  BDFID:                   0
  Internal Node ID:        0
  Compute Unit:            16
  SIMDs per CU:            0
  Shader Engines:          0
  Shader Arrs. per Eng.:   0
  WatchPts on Addr. Ranges:1
  Memory Properties:
  Features:                None
  Pool Info:
    Pool 1
      Segment:                 GLOBAL; FLAGS: FINE GRAINED
      Size:                    32780072(0x1f42f28) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:4KB
      Alloc Alignment:         4KB
      Accessible by all:       TRUE
    Pool 2
      Segment:                 GLOBAL; FLAGS: KERNARG, FINE GRAINED
      Size:                    32780072(0x1f42f28) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:4KB
      Alloc Alignment:         4KB
      Accessible by all:       TRUE
    Pool 3
      Segment:                 GLOBAL; FLAGS: COARSE GRAINED
      Size:                    32780072(0x1f42f28) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:4KB
      Alloc Alignment:         4KB
      Accessible by all:       TRUE
  ISA Info:
*******
Agent 2
*******
  Name:                    gfx1031
  Uuid:                    GPU-XX
  Marketing Name:          AMD Radeon RX 6700 XT
  Vendor Name:             AMD
  Feature:                 KERNEL_DISPATCH
  Profile:                 BASE_PROFILE
  Float Round Mode:        NEAR
  Max Queue Number:        128(0x80)
  Queue Min Size:          64(0x40)
  Queue Max Size:          131072(0x20000)
  Queue Type:              MULTI
  Node:                    1
  Device Type:             GPU
  Cache Info:
    L1:                      16(0x10) KB
    L2:                      3072(0xc00) KB
    L3:                      98304(0x18000) KB
  Chip ID:                 29663(0x73df)
  ASIC Revision:           0(0x0)
  Cacheline Size:          128(0x80)
  Max Clock Freq. (MHz):   2725
  BDFID:                   2048
  Internal Node ID:        1
  Compute Unit:            40
  SIMDs per CU:            2
  Shader Engines:          2
  Shader Arrs. per Eng.:   2
  WatchPts on Addr. Ranges:4
  Coherent Host Access:    FALSE
  Memory Properties:
  Features:                KERNEL_DISPATCH
  Fast F16 Operation:      TRUE
  Wavefront Size:          32(0x20)
  Workgroup Max Size:      1024(0x400)
  Workgroup Max Size per Dimension:
    x                        1024(0x400)
    y                        1024(0x400)
    z                        1024(0x400)
  Max Waves Per CU:        32(0x20)
  Max Work-item Per CU:    1024(0x400)
  Grid Max Size:           4294967295(0xffffffff)
  Grid Max Size per Dimension:
    x                        4294967295(0xffffffff)
    y                        4294967295(0xffffffff)
    z                        4294967295(0xffffffff)
  Max fbarriers/Workgrp:   32
  Packet Processor uCode:: 122
  SDMA engine uCode::      80
  IOMMU Support::          None
  Pool Info:
    Pool 1
      Segment:                 GLOBAL; FLAGS: COARSE GRAINED
      Size:                    12566528(0xbfc000) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:2048KB
      Alloc Alignment:         4KB
      Accessible by all:       FALSE
    Pool 2
      Segment:                 GLOBAL; FLAGS: EXTENDED FINE GRAINED
      Size:                    12566528(0xbfc000) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:2048KB
      Alloc Alignment:         4KB
      Accessible by all:       FALSE
    Pool 3
      Segment:                 GROUP
      Size:                    64(0x40) KB
      Allocatable:             FALSE
      Alloc Granule:           0KB
      Alloc Recommended Granule:0KB
      Alloc Alignment:         0KB
      Accessible by all:       FALSE
  ISA Info:
    ISA 1
      Name:                    amdgcn-amd-amdhsa--gfx1031
      Machine Models:          HSA_MACHINE_MODEL_LARGE
      Profiles:                HSA_PROFILE_BASE
      Default Rounding Mode:   NEAR
      Default Rounding Mode:   NEAR
      Fast f16:                TRUE
      Workgroup Max Size:      1024(0x400)
      Workgroup Max Size per Dimension:
        x                        1024(0x400)
        y                        1024(0x400)
        z                        1024(0x400)
      Grid Max Size:           4294967295(0xffffffff)
      Grid Max Size per Dimension:
        x                        4294967295(0xffffffff)
        y                        4294967295(0xffffffff)
        z                        4294967295(0xffffffff)
      FBarrier Max Size:       32
*** Done ***
```

{{% /details %}}

If you get mostly empty output, or it lists only your CPU as an agent, it means
that your GPU is not recognised as a ROCm platform. You can check and see if
there is unofficial support somewhere, but it is unlikely.

### Priming ROCm on Fedora

One last thing about ROCm that needs to be addressed on Fedora is that Fedora
packages use directories that are not standard according to the ROCm project.
This can and likely will cause problems when trying to set up AI tools, without
it being obvious why.

By standard, ROCm development files come with copies of LLVM tools like clang
and clang++ in `/opt/rocm`, which is what the tools expect. On Fedora, however,
the ROCm packages do not do that. By running `hipcc --version` you can find out
where the LLVM tools are, which should be located at `/usr/lib64/llvm18/bin`.

> [!IMPORTANT]
> The `llvm18` part can change. Make sure you check **your** version first!

Because of this, you need to make symbolic links:
- `/opt/rocm/llvm/bin/clang` to `/usr/lib64/llvm18/bin/clang`
- `/opt/rocm/llvm/bin/clang++` to `/usr/lib64/llvm18/bin/clang++`

This can be done with the following snippet:
```fish
sudo mkdir -p /opt/rocm/llvm &&
sudo ln -s /usr/lib64/llvm18/bin/clang /opt/rocm/llvm/bin/clang &&
sudo ln -s /usr/lib64/llvm18/bin/clang++ /opt/rocm/llvm/bin/clang++
```

Now you should not be hitting into mysterious snags the way I have, as now
there will be an `/opt/rocm` populated with the libraries and tools that are
expected.

### Vulkan and CLBlast

It is not the end of the world if your GPU is not supported by ROCm, because it
still can be supported by Vulkan or CLBlast. Vulkan is the same API that is
used to play your favourite games on Steam on Fedora, and it can be used to do
AI workloads as well. It is just not nearly as performant as ROCm. However, it
is very much an attractive option, and still beats CPU-only inference by orders
of magnitude.

You will need to install the following packages:

```fish
sudo dnf install mesa-libOpenCL-devel libclc-devel clinfo
```

Similarly with ROCm, you can verify if your GPU is Vulkan-enabled by running
`clinfo`. If you get a long list (as in the ROCm case above), then it means
your GPU is recognised as a valid Vulkan platform. If you don't see much, then
it is not supported, but it really should be, unless you have *exceptionally*
old or cheap hardware.

One thing to note is that Vulkan requires your CPU to have the AVX2 extension.
This should be present on all modern CPUs, unless, as said, you have
exceptionally old or cheap hardware. Should you not have AVX2 on your CPU for
whatever reason, CLBlast is one last option that might be supported on your
hardware, as it requires AVX instead. It requires the same libraries and tools
as above, so you don't need anything more. Just make sure to select a "CLBlast
option" or "OpenCL option," if one exists, whenever you use your AI tool of
choice.

If your CPU does not even have the AVX extension, then you probably need to
upgrade your hardware. However, it is worth nothing that [version 1.82.4 of
koboldcpp](https://github.com/LostRuins/koboldcpp/releases/tag/v1.82.4) no
longer requires AVX for its CLBlast backend, so your toaster can now run it.

## Conclusion

![Low resolution screenshot showing ComfyUI in action](/ai_on_fedora/comfyui.webp)

If this guide has helped at least one person, then my mission is complete. The
information contained here took me at least a dozen hours of troubleshooting
and frustration to gather -- probably more. The worst part is that some of it
is still a bit unsure. My hope is that this page can become a small repository
of Fedora-specific knowledge that other Fedora users can use to help each
other, since getting AI to work on Fedora can be quite frustrating.

Leave a comment below, or place an issue / pull request [on
GitHub](https://github.com/docs-mj/docs-mj.github.io), if you want to share
something not covered here, or, better yet, correct discrepancies.
