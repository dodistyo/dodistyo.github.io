---

title: "SeedVR2 + HYPIR: The Photo Restoration Duo I Never Expected to Be This Good"
subtitle: "A good model combination for photo restoration"
date: 2026-08-14T09:00:00+07:00
lastmod: 2026-08-15T09:00:00+07:00
draft: false
author: "Dodi Prasetyo"
description: "I combined SeedVR2 and HYPIR in ComfyUI to restore an old faded family photo. The results beat proprietary tools. Here is the workflow and why the model order matters."
tags: ["AI", "ComfyUI", "Photo Restoration", "Open Source", "SeedVR2", "HYPIR", "Image Upscaling", "Intermediate"]
categories: ["Artificial Intelligence"]
weight: 1
resources:
 - name: "featured-image"
   src: "featured-image.png"
---

**I have this faded family photo that has been sitting in a drawer for years.**


You know the kind. The colors are washed out, the details are blurry, and every time you look at it, you wish you could see the faces more clearly. For a long time, I thought the only option was to find a restoration service or use one of those online tools that promise miracles.


I've been in the AI space long enough to know things move fast, but my eyes were on the LLM race. Image models? I wasn't keeping up.
Then I found two open-source models that most people probably don't even know exist.

SeedVR2 from ByteDance and HYPIR from XPixel Group. Both are image restoration models, both are free, and both run locally. What I didn't expect was that chaining them in the right order would produce results that beat the proprietary tools I was comparing against.

Here is what I built, how it works, and why the order of operations actually matters.

---

## The Photo

{{< image src="original-photo.jpg" alt="Original faded family photo" caption="The original faded family photo I used as test material" >}}

I picked an old family photo that had seen better days. The scan was low resolution, the colors had faded, and the facial details were barely readable. Perfect test material for a restoration workflow.

The goal was simple: make it look like the photo was supposed to look when it was new, without hallucinating details that were never there.

---

## Meet The Models

### SeedVR2

SeedVR2 is a one-step diffusion-based restoration model from ByteDance's Seed research team. It was published at ICLR 2026 (model weights released June 2025) and is designed primarily for video restoration, but it works exceptionally well on images too.

What makes it interesting is the adversarial training approach. Instead of the typical multi-step diffusion process that takes dozens of denoising steps, SeedVR2 does it in one shot. The adversarial component helps it preserve structural integrity while adding detail. You get sharper edges, better color recovery, and less of that smudged look that some upscalers produce.

The 3B parameter model runs on a single GPU and handles both image and video frames. It is available on Hugging Face under the ByteDance-Seed organization.

### HYPIR

HYPIR takes a different approach. It is built on Stable Diffusion 2.1 (released July 2022) and uses diffusion-yielded score priors to guide the restoration process. The model comes from XPixel Group (arXiv July 2025), a collaboration between SIAT, CUHK, HKMU, INSaIT, and SUAT.

Where SeedVR2 excels at global structure and color, HYPIR shines at local texture recovery. It can recover skin texture, fabric grain, and fine details that other models tend to smooth over. The model also supports text prompts to guide the restoration direction, which gives you control over how aggressive the enhancement should be.

HYPIR outputs at resolutions above 2K and uses a tiling strategy to handle large images without running out of memory.

---

## Why Combine Them?

On paper, these models solve the same problem. In practice, they bring different strengths to the table.

SeedVR2 handles the big picture. It recovers the overall structure, fixes color casts, and upscales to a usable resolution. But it doesn't always nail the fine textures.

HYPIR handles the details. It refines texture, sharpens edges at a micro level, and adds the kind of grain and noise that makes a photo look natural instead of artificially smooth.

Put them together in the right order, and you get something better than either model alone.

---

## The Order Matters

This is the part I had to figure out through trial and error.

I tested both workflows:

1. **HYPIR first, then SeedVR2** - restore details first, then upscale
2. **SeedVR2 first, then HYPIR** - upscale first, then restore details

The second order won. Here is why.

When you run HYPIR on a low-resolution photo, it works with limited pixel information. The model has to guess at details from a small canvas. Then when SeedVR2 upscales, it stretches those guesses. The result looks okay at first glance, but the details feel artificial and the texture doesn't hold up under scrutiny.

When you run SeedVR2 first, it upscales the photo to a higher resolution while preserving the original structure. Now HYPIR has more pixels to work with. The model can recover texture and detail from a larger, cleaner canvas. The result is sharper, more natural, and holds up better when you zoom in.

I ran the same photo through both orders. The SeedVR2-first pipeline produced noticeably better results, especially in facial features and fabric texture.

> 💡 **The lesson**: upscale the structure first, then refine the details. Let HYPIR work on higher-resolution pixels, not on guesses from a tiny input.

---

## The Workflow

Here is what the ComfyUI setup looks like:

{{< image src="workflow-used.png" alt="ComfyUI workflow showing the SeedVR2 pipeline" caption="ComfyUI workflow showing the SeedVR2 first, HYPIR second pipeline" >}}

The workflow loads the photo, runs it through SeedVR2 for upscaling and structural recovery, then passes the output to HYPIR for texture refinement and detail enhancement. The sidebar shows both workflows in the image enhancement category.

I won't go node by node here. The graph speaks for itself if you are familiar with ComfyUI. If you want the exact workflow setup with all the parameters, leave a comment and I will share it.

---

## The Results

Here is the full before and after:

{{< image src="original-vs-enhanced-(seedvr-hypir).png" alt="Full comparison of original faded photo versus SeedVR2 + HYPIR enhanced version" caption="Full comparison: original faded photo versus SeedVR2 + HYPIR enhanced version" >}}

The color recovery alone is worth it. The faded sepia tones are gone, the faces are readable, and the overall image looks like it was scanned properly instead of photocopied three times.

Now let us zoom in on the details:

{{< image src="original-vs-enhanced-zoomed-(seedvr-hypir).png" alt="Zoomed comparison showing texture recovery and facial detail" caption="Zoomed comparison showing texture recovery and facial detail" >}}

The texture work is where HYPIR earns its place in the pipeline. Skin texture, fabric grain, background detail - all of it is recovered without looking artificially sharpened. The model knows when to add detail and when to leave things alone.

---

## How Does It Compare to Proprietary?

I ran the same photo through a few proprietary restoration services to see how this open-source combo stacks up.

{{< image src="output-comparison-with-proprietary.png" alt="Comparison of SeedVR2 + HYPIR output against proprietary restoration tools" caption="Comparison of SeedVR2 + HYPIR output against proprietary restoration tools" >}}

Each panel shows which proprietary tool it is compared against. Nano Banana and ChatGPT both do decent work. Nano Banana still looks a bit blurry. ChatGPT produces good quality, but the facial identity shifts - the face changes just enough to feel like someone else. Look closely and decide for yourself.

What matters most for me: the faces are still recognizable. The image is clear, the details are there, and the facial identity is preserved. That is what restoration should do.

## Hardware

You will need a decent GPU to run this pipeline comfortably. I tested on a setup with **24GB VRAM**, a multi-core CPU, and enough RAM to handle the model loading without swapping.

SeedVR2 3B is the heavier component. HYPIR is lighter but benefits from having headroom for the tiling strategy. If you have less VRAM, you can still run it with some parameter adjustments, but expect longer processing times and potentially lower quality on larger images.

---

## Why This Matters

The open-source image restoration space is moving fast. Models like SeedVR2 and HYPIR aren't just catching up to proprietary tools. In some cases, they are passing them.

What this workflow shows is that you don't need a subscription or an API key to get professional-quality photo restoration. You need a good GPU, some patience with ComfyUI, and the right combination of models.

The fact that the order matters is the kind of insight you only get by running the models yourself. Reading about them doesn't tell you that SeedVR2-first produces better texture recovery. You have to test it.

---

## What Next?

If you want to try this yourself, the models are available:

- **SeedVR2**: [ByteDance-Seed/SeedVR](https://github.com/ByteDance-Seed/SeedVR) on GitHub, models on Hugging Face
- **HYPIR**: [XPixelGroup/HYPIR](https://github.com/XPixelGroup/HYPIR) on GitHub, with a live demo at [hypir.xpixel.group](https://hypir.xpixel.group/)

Both have ComfyUI custom nodes available. Install them, load the models, and experiment with the order. You might be surprised what you can restore from that old photo in your drawer.

If you want the exact workflow setup I used here, drop a comment and I will share it.

> 💡 **Final thought**: The best restoration pipeline isn't necessarily the most expensive one. It is the one that combines the right models in the right order. Sometimes the answer is two free models chained together in ComfyUI.
