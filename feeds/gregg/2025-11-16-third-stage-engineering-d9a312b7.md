---
title: Third Stage Engineering
url: http://www.brendangregg.com/blog//2025-11-17/third-stage-engineering.html
published: "2025-11-16T13:00:00Z"
feed: gregg
guid: http://www.brendangregg.com/blog//2025-11-17/third-stage-engineering.html
---

# Third Stage Engineering

The real performance of any computer hardware in production is the result of the hardware, software, and tuning; the investment and sequence of these efforts can be pictured as a three-stage rocket:

![](/blog/images/2025/3rd_stage_engineering.png)

I recently presented this embarrassingly simple diagram to Intel's executive leadership, and at the time realized the value of sharing it publicly. The Internet is awash with comparisons about Intel (and other vendors’) product performance based on hardware performance alone, but the performance of software and then tuning can make a huge difference for your particular workload. You need all three stages to reach the highest, and most competitive, performance.

It's obvious why this is important for HW vendors to understand internally – they, like the Internet, can get overly focused on HW alone. But customers need to understand it as well. If a benchmark is comparing TensorFlow performance between HW vendors, was the Intel hardware tested using the [Intel Extension for TensorFlow Software](https://github.com/intel/intel-extension-for-tensorflow), and was it then tuned? The most accurate and realistic evaluation for HW involves selecting the best software and then tuning it, and doing this for all HW options.

I spend a lot of time on the final stage, tuning – what I call _third-stage engineering_. It's composed of roughly four parts: People, training, tools, and capabilities. You need staff, you need them trained to understand performance methodologies and SW and HW internals, they need tools to analyze the system (both observational and experimental), and finally they need capabilities to tune (tunable parameters, settings, config, code changes, etc.).

I see too many HW evaluations that are trying to understand customer performance but are considering HW alone, which is like only testing the first stage of a rocket. This doesn't help vendors or customers. I hope that's what my simple diagram makes obvious: We need all three stages to reach the highest altitude.
