---
title: "Coding Agents: DeepSeek V4 vs. Codex"
date: 2026-05-23 10:10:00 +0800
categories: [AI for Coding]
tags: [Android, GPU, DeepSeek, DeepSeek V4, DeepCode, OpenAI, Codex, Codex CLI]
pin: true
---

# Introduction

As an engineer with experience in graphics application development, Android cloud gaming, GPU virtualization, and AI accelerator software stacks, I recently started a side project: building an Android GPU benchmark application.

The project became an opportunity to combine two worlds — my long-term experience in GPU and Android systems, and my newer exploration of AI-assisted development. What made the experiment particularly interesting was not only whether modern coding agents could complete the project, but also *how differently they approached the same engineering problem*.

To explore this, I developed similar versions of the application using two AI coding agents:

- OpenAI’s Codex CLI
- DeepSeek V4 through DeepCode in VS Code

Although both systems ultimately delivered working solutions, their “engineering personalities” turned out to be remarkably different.

---

# Working with Codex CLI

My day job provides access to Codex CLI as an AI-assisted development tool. While powerful, it operates within a fairly controlled workflow, especially when dealing with Java and Gradle environments. In practice, I often needed to execute build commands manually outside the sandbox.

The project started from a standard Android Studio “Game Activity” template. After that, I handed the implementation almost entirely to Codex with a simple prompt:

> “This is an Android Game Activity project. Please design and develop an Android GPU benchmark application.”

Once the OpenGL ES framework was functioning, I asked Codex to extend the application with Vulkan benchmarking support.

What stood out immediately was Codex’s structured engineering process. Rather than aggressively modifying files, it behaved like a careful senior engineer working within a controlled production environment:

- proposing milestones,
- documenting decisions,
- presenting source-code patches,
- requesting approval before edits,
- asking permission to compile or run builds,
- maintaining `README.md` and `agent.md`,
- and finally generating commit messages after validation.

One particularly impressive moment was Codex’s Vulkan integration strategy. Instead of immediately replacing the existing GLES framework, it proposed an eight-step migration plan:

1. Refactor the rendering abstraction layer  
2. Introduce a minimal Vulkan framework  
3. Validate swapchain creation  
4. Add clear-color rendering  
5. Add synchronization primitives  
6. Introduce workload support  
7. Validate timing infrastructure  
8. Gradually expose Vulkan benchmarking  

Codex also recommended keeping Vulkan functionality dormant behind an Android property switch until the implementation had been fully validated. This prevented architectural conflicts between GLES and Vulkan from appearing later in development.

Equally notable was that Codex independently designed two benchmark workloads targeting:

- vertex throughput
- fragment/pixel processing performance

The entire experience felt methodical, stable, and thoughtfully engineered — closer to collaborating with a cautious platform architect than a rapid prototype engineer.

Overall, the project progressed smoothly over roughly eight hours.

---

# Working with DeepSeek V4

After finishing the Codex version, I immediately wanted to try the same project using DeepSeek V4 through DeepCode in VS Code.

Using the exact same initial prompt, the difference in behavior became obvious almost instantly.

Where Codex behaved cautiously and incrementally, DeepSeek V4 acted aggressively and execution-focused. Instead of proposing a conservative roadmap, it immediately generated a broad benchmarking strategy with five workloads covering:

- texture fill rate
- texture bandwidth
- vertex throughput
- shader ALU throughput
- mixed real-world rendering scenarios

(Appendix A contains the full workload table.)

Its development style also differed significantly:

- directly editing files,
- repeatedly compiling,
- self-correcting build failures,
- continuing implementation autonomously,
- and evolving the codebase toward milestones with minimal intervention.

In contrast to Codex, documentation was largely ignored unless explicitly requested. The focus was overwhelmingly on implementation speed and feature completion.

The difference became even more visible during Vulkan integration.

Codex proposed an eight-step staged migration. DeepSeek V4 proposed essentially a two-step approach:

1. Add Vulkan rendering support  
2. Add Vulkan benchmarking workloads  

This accelerated implementation dramatically, but also introduced architectural oversights. For example, DeepSeek initially failed to properly manage switching between GLES and Vulkan contexts. When Vulkan support was added, GLES benchmarking functionality unintentionally became disabled.

Codex had anticipated this issue during the design phase and proposed a dedicated Android property mechanism for backend selection.

After I pointed out the problem, however, DeepSeek adapted quickly and implemented a simpler command-line-based backend switch:

```bash
am start -n com.example.gpubench/.MainActivity --es api gles
am start -n com.example.gpubench/.MainActivity --es api vulkan
```

In some areas, DeepSeek demonstrated stronger practical instincts.

For Vulkan shaders, DeepSeek proposed two alternative approaches and ultimately embedded shader binaries directly into header files. This avoided APK resource-packaging issues that later appeared in the Codex implementation, where shader resources were not correctly included during device deployment.

Benchmark output formatting also reflected the personality difference between the systems.

Codex produced highly structured machine-readable output:

- JSON-friendly
- script-oriented
- information-dense

DeepSeek instead emphasized human readability:

- aligned formatting
- visually prominent metrics
- easier manual inspection

The cost difference was also striking.

The entire DeepSeek project cost less than approximately $1 in API usage, despite consuming an estimated 30–40 million tokens (with heavy cache reuse). The downside was occasional service instability — twice the session terminated unexpectedly and required several hours before resuming.

Codex, by comparison, remained consistently stable throughout development, though pricing visibility was unavailable in my environment.

---

# Summary Comparison

| Category | DeepSeek V4 | Codex | Advantage |
|---|---|---|---|
| Android Game Activity startup | Strong | Strong | Tie |
| Benchmark workloads proposed | 5 workloads | 2 workloads | DeepSeek V4 |
| Coding workflow | Autonomous editing and iterative fixing | Patch-based with approvals | Depends on workflow |
| Documentation | Minimal unless requested | Continuously maintained | Codex |
| Milestone planning | Lightweight | Structured and explicit | Codex |
| AVD testing | Good | Good | Tie |
| Real-device testing | Good | Restricted by policy | DeepSeek V4 |
| Vulkan integration strategy | Fast 2-step approach | Structured 8-step roadmap | Codex |
| Shader/resource handling | Robust embedded shader approach | Resource packaging issue appeared | DeepSeek V4 |
| Graphics backend switching | Initially overlooked | Designed early | Codex |
| Benchmark output formatting | Human-readable | Machine-oriented | DeepSeek V4 |
| Cost | Extremely low | Unknown | DeepSeek V4 |
| Service stability | Occasional interruptions | Stable | Codex |

---

# Conclusion

Both systems successfully delivered a functional Android GPU benchmark application, but they approached the task with fundamentally different engineering philosophies.

DeepSeek V4 behaved like an experienced engineer receiving an urgent 7 PM request to prepare a demo for the next morning:

- fast,
- aggressive,
- implementation-driven,
- highly autonomous,
- occasionally rough around the edges,
- but remarkably effective under time pressure.

Codex, in contrast, behaved more like a platform engineer responsible for a maintainable production system:

- structured,
- cautious,
- documentation-oriented,
- architecture-aware,
- process-driven,
- and highly reliable.

Neither approach is universally better. Their strengths align with different development scenarios:

- DeepSeek excels at rapid prototyping and feature acceleration.
- Codex excels at maintainability, process discipline, and long-term architecture.

The most interesting realization from this experiment was that modern coding agents are no longer distinguished only by model capability — they are increasingly differentiated by *engineering personality*.

---

# Appendices

## Appendix A — Workloads Proposed by DeepSeek V4

| Workload | Geometry | Fragment Shader | What It Tests |
|---|---|---|---|
| **Fill Rate / Overdraw** | N fullscreen quads (N=4, 8, 16, 32) drawn back-to-front with alpha blending | Simple texture sample | Pixel fill rate under increasing overdraw |
| **Vertex Throughput** | Procedural grid or sphere, configurable triangle count (5k–500k) | Pass-through (solid color) | Vertex processing + triangle setup |
| **Texture Bandwidth** | One fullscreen quad | Multiple texture samples per pixel (4, 8, 16 taps from a large texture) | Texture cache + memory bandwidth |
| **ALU Compute** | One fullscreen quad | Procedural noise (e.g., 3D simplex noise) or a Blinn-Phong lighting loop | Shader ALU throughput |
| **Mixed** | Several 3D meshes (cube, sphere) with rotation | Textured + Phong lighting with normal maps | Representative real-world balance |

---

## Appendix B — Output of Codex Solution

```text
GPUBench result backend=vulkan pass=vertex_shader_instanced_scene metric_source=vulkan_timestamp_query frames=120 avg_ms=0.052 min_ms=0.050 max_ms=0.054 p50_ms=0.052 p90_ms=0.053 p95_ms=0.053 stddev_ms=0.001 fps=19237.478 cpu_avg_ms=0.160 gpu_avg_ms=0.052 missing_gpu_frames=0 workload="instances=25 sphere_vertices=8385 triangles=409600 loader_available=true instance_created=true surface_created=true physical_device_selected=true device_created=true swapchain_created=true swapchain_images=4 framebuffers=4 command_buffers=4 timestamp_query=true khr_surface_extension=true swapchain_extension=true android_surface_extension=true swapchain_extent=1280x720 swapchain_format=37 instance_version=1.1.0"

GPUBench result_json {"backend":"vulkan","pass":"vertex_shader_instanced_scene","metric_source":"vulkan_timestamp_query","width":1280,"height":720,"warmup_frames":30,"measured_frames":120,"primary_frames":120,"primary_avg_ms":0.052,"primary_min_ms":0.050,"primary_max_ms":0.054,"primary_p50_ms":0.052,"primary_p90_ms":0.053,"primary_p95_ms":0.053,"primary_stddev_ms":0.001,"primary_fps":19237.478,"cpu_frames":120,"cpu_avg_ms":0.160,"gpu_frames":120,"gpu_avg_ms":0.052,"gpu_missing_frames":0,"workload":"instances=25 sphere_vertices=8385 triangles=409600 loader_available=true instance_created=true surface_created=true physical_device_selected=true device_created=true swapchain_created=true swapchain_images=4 framebuffers=4 command_buffers=4 timestamp_query=true khr_surface_extension=true swapchain_extension=true android_surface_extension=true swapchain_extent=1280x720 swapchain_format=37 instance_version=1.1.0"}
```

---
The Codex output is comprehensive and highly script-friendly, but optimized primarily for machine parsing rather than human readability.

## Appendix C — Output of DeepSeek V4 Solution

```text
========================================
[Vulkan Fill Rate] warmup complete, measuring...
========================================
BENCHMARK: Vulkan Fill Rate
Frames:     120
Min:        656.072 us
Max:        679.732 us
Avg:        671.755 us  (1488.64 fps)
Median:     671.692 us
========================================
```

The DeepSeek output emphasizes readability and quick visual inspection, making it easier to evaluate benchmark behavior interactively.

## Appendix D — Repo of DeepSeek Solution

[https://github.com/baifang/AndroidGPUBench](https://github.com/baifang/AndroidGPUBench)
