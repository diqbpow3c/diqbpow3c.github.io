---
layout: default
title: Advanced techniques for improving the performance of large-scale LLM serving systems
nav_order: 3
feed: true
date: 2026-01-20
---

# Advanced techniques for improving the performance of large-scale LLM serving systems


{: .fs-6 .fw-300 }

{% if page.date %}
  <time datetime="{{ page.date | date_to_xmlschema }}">
    {{ page.date | date: "%Y-%m-%d" }}
  </time>
{% endif %}

# Intro
Many companies deploy their own LLM inference service for regulatory compliance, data security, privacy, customization and other reasons. 

Assuming that you are already using vLLM, SGLang, or other frameworks, and are familiar with the commonly known performance optimization techniques like CUDA graph, here you can find some advanced techniques and useful information about improving the performance of the inference systems. 

# The performance bottlenecks
The first thing to do is to **make sure you know what metrics you want to optimize**. You might be trying to optimize the TTFT (time to first token), throughput, throughput under some SLA, or others.

Then, you could find out what the bottlenecks are. Common Bottlenecks include
- KV cache memory pressure
- GPU underutilization
- Network overhead
- Load imbalance

You can’t optimize what you don’t measure.

# Advanced optimization techniques
## (Often overlooked) Quantization
I've seen some companies setting up a large GPU cluster, using many optimizations, but neglecting one of the **most important** optimization that can be easily achieved: using a quantized model. Switching to a quantized model, along with libraries like flashattention, could significantly improve the performance. A rough estimation is that, by switching from FP16 to FP8, you get 1.2× – 1.6× higher throughput, 15%–30% lower latency, with a negligeable approximately 1% drop in accuracy (quality).

## Speculative decoding
Speculative decoding is essentially using a smaller model to generate drafts, and using the large model for verification. Although ChatGPT and Gemini do not publicly confirm whether they are using speculative decoding, but this technology is easy to implement and gives your a considerable performance boost.

## Prefill-decode disaggregation
This technique has been adopted by large companies like Meta. The computational characteristics of the prefilling and decoding are different, and this leads to an opportunity to improve the performance. vLLM and SGLang already support this functionality.

**The cold-start issue and autoscaling for P-D instances**. The time consumed for starting a new vLLM or SGLang instance could reach up to several minutes. Even the initialization process of inference engines take tens of seconds, depending on the model size and other factors. That means you may want to add traffic prediction models, in order to proactively scale the servers. Additionally, according to a blog by Meta, it might be a good idea to automatically decide the ratio between prefilling and decoding instances. The NVIDIA [Dynamo Planner](https://docs.nvidia.com/dynamo/latest/planner/planner_intro.html) solves these two problems simultaneously. The source code is available and you can check it out. 

## Distributed KV cache 
The performance of LLM serving systems are often constrained by the limited size of KV cache on your GPU. You can use hierarchical and distributed KV cache to increase the size, and significantly reduce latency while improving throughput. The KV cache is stored and shared across machines, and can reside on GPU memory, CPU memory, and even external SSDs. You could use the [Mooncake](https://github.com/kvcache-ai/Mooncake) library for this purpose, which integrates well with SGLang. 

## Request routing
Since you are running a cluster, request routing becomes a major issue. Routing decisions have to be made considering factors like length of queues, KV cache availability, and others. Luckily, there are libraries like SGLang-router which can help you solve this problem. 

## Scheduling policy
By default, vLLM uses the first-come-first-serve policy for incoming requests, and SGLang utilizes a highly optimized, cache-aware scheduling policy designed to maximize GPU utilization and minimize KV cache eviction, particularly when handling multi-turn conversations and structured prompts. Its scheduler operates by analyzing the RadixAttention tree to prioritize requests that share the longest prefix with currently cached data.

If you are aiming to optimize certain metrics, like goodput (throughput that satisfy the SLA), you could change the scheduling policy and find out whether it helps. 

## SGLang RBG
If you are using Kubernetes to manager the cluster, and find it difficult to manage the LLM-specific topology, like PD-disaggregation nodes, and dedicated KV cache nodes, you could try [SGLang RBG](https://github.com/sgl-project/rbg), which is a Kubernetes API for orchestrating distributed, stateful AI inference workloads with multi-role collaboration and built-in service discovery.

