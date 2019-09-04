---
title: "BPX 4 Hyperledger"
description: "How to use the Caterpillar engine to execute business processes on Hyperledger Fabric"
author: Gowtham Moham, Timotheus Kampik
tags: Blockchain, Hyperledger, Ethereum, Business Process Execution
layout: article
image:
  feature: /2019/web.jpg
  alt: "Web"
---

In this blog post, we show how to execute BPMN 2.0 XML diagrams on Hyperledger Fabric, using the open source tool *Caterpillar*.

## BPX on the Blockchain with Caterpillar
The intersection of blockchain technology and business process management has recently received increased attention by the academic community.
One of the research results is the [Caterpillar](https://github.com/orlenyslp/Caterpillar) engine, a prototype that can compile BPMN 2.0 XML diagrams (with some restrictions) to Solidity smart contracts and execute them on the Ethereum blockchain.
This means, Caterpillar allows business users to specify smart contracts graphically as flowchart-like *BPMN* diagrams; now, you don't need to be a coder to write smart contracts!

****************
<p align="center">A sample BPMN diagram which can be deployed as smart contract</p>

![alt sample](https://github.com/signavio/tech-blog/blob/caterpillar-hyperledger/assets/img/BPMNSampleDIagram.png)
****************


## Why Hyperledger Fabric matters
However, by default, Caterpillar integrates with Ethereum (and the Ethereum [Ganache client](https://www.trufflesuite.com/ganache)), which is problematic because Ethereum is typically not considered suitable for Enterprise use cases:

* Ethereum's transaction costs and price fluctuations add unwanted volatility to the business environment.

* The association with *snake oil* schemes and ICOs, as well as the unchecked influence of one individual project lead decrease the willingness of serious enterprises to belief in the long-term sustainability of the ecosystem.

In contrast, the [Hyperledger Fabric](https://www.hyperledger.org/projects/fabric) framework implements a [consortium blockchain](https://hedgetrade.com/blockchain-consortium/) approach: it limits network participation to authenticated parties that are willing to participate in consensus-finding without additional (on-chain) financial incentives.
This makes cryptocurrency features and transaction costs obsolete. 

Also, Hyperledger is backed by well-known industry giants like SAP and IBM, and managed by the Linux Foundation, which provides a comparably high degree of institutional trust.

## Getting it running

Technical tutorial

## Conclusion
As we have shown in this blog post, executing BPMN diagrams on Hyperledger Fabric is relatively straight-forward after making minor adjustments to Caterpillar.
Still, we think that more work is necessary to allow for the business user-friendly specification and deployment of chain code:

**Test-driven chain code specification**  
  Considering the software development best practice of *test driven* development, process modelers should be able to specify test cases that describe the target behavior directly in their modeling environment.

**Continuous integration**
  The process of chain code modeling, testing and deployment should be supported by continuous integration pipelines.


<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@clintadair?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Clint Adair"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Clint Adair</span></a>
