---
title: "Load Balancing Omnissa Horizon Connection Servers"
date: 2026-01-02
categories: [Omnissa, Horizon]
---

## Overview
In this post, I document how I load balanced three VMware Horizon
Connection Servers in a production-style environment.
<img width="1200" height="220" alt="image" src="https://github.com/user-attachments/assets/95009809-8db3-433f-b364-7c240242dce1" />

## Architecture
- 3 × Horizon Connection Servers
- External Load Balancer
- SSL offloading considerations

## Key Learnings
- Persistence is critical
- Health monitors must check `/broker/xml`
