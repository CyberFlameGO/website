---
title: Moracle development update
description: Learnings from the past year and a new approach to meeting the data needs of blockchain applications
date: 2021-03-27
tags:
  - lisk
  - open-source
layout: layouts/post.njk
eleventyExcludeFromCollections: true
---

## What is Moracle?
Moracle is a <a href="https://academy.binance.com/en/articles/blockchain-oracles-explained" target="_blank">decentralized software oracle</a> (click this link for a definition) built specifically for the <a target="_blank" href="https://lisk.io/sdk">Lisk SDK</a>.


Developers using the Lisk SDK face the following problems:
1. They cannot access external internet APIs within their applications.
2. It's difficult to large amounts of structured data in the user asset data fields provided by the SDK. This makes building more complicated applications such as a social networks or marketplaces difficult.
3. Sending assets between chains is extremely difficult. This is particularly important because each Lisk application has its own blockchain.

The project's goal is to solve these problems by giving developers the following capabilities:
1. Access internet APIs from their blockchain applications. Eg. checking an asset price, state of an IoT device, or getting information from a centralized platform like GitHub. This is hence referred to as "oracle functionality". 
2. Store larger amounts of information in an SQL database synced across all nodes.
3. Allow developers to implement cross-chain asset swaps by utilizing an oracle to verify transactions on a foreign chain. This will allow, for example, NFTs issued on a custom blockchain to be securely purchased using the common Lisk token. 

## The story of Moracle so far
I've been involved with the Lisk community since the summer of 2017. I published the first Moracle whitepaper in late 2017 and received a lot of helpful feedback from the community. During this time, I partnered with [StellarDynamic](https://sidechainsolutions.io/) and he was incredible in supporting development and promotion of the project. A proof-of-concept notary service was released in December of 2018.

During 2019, I discovered GraphQL while working at <a target="_blank" href="https://halp.com">Halp</a> and decided it was the best way to build an oracle service that delivers structured data in a reliable format. I decided to completely rewrite Moracle with a new design and strategy. The vision was to combine every data source into a single GraphQL schema, allowing developers to fetch information by embedding GraphQL queries in their transaction implementations. I submitted a [demo](https://www.youtube.com/watch?v=S4roWDIJRlM) and then presented Moracle at Lisk.js 2019 (embedded below).

<div class="captionedimg">
<iframe style="width: 90%" height="300" src="https://www.youtube.com/embed/_LjfZMRuCgU" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe><br>
</div>

After attending Lisk.js, I received the [Lisk Builders' Program](https://lisk.io/builders-program) grant and have continued to work on Moracle throughout 2020. During this time, I did some contracting work for a company which wanted to use blockchain and was interested in using Moracle; they ended up pivoting to a centralized platform rather than a blockchain one, but it was a valuable experience regardless.

The project has experienced some delays related to to my college course load and the pandemic, but I'm happy to announce that a robust oracle solution for the Lisk SDK will be here soon!

## New features
Moracle services now have __enforced return types__. This allows service validation logic to assume certain properties of a return value safely. These types are:
<table class="prettytable">
<tr>
<th>Type Name</th>
<th>Description</th>
</tr>
<tr>
<td>JSON</td>
<td>Most common, used for generic nested data objects of unknown ahead-of-time structure.</td>
</tr>
<tr>
<td>Number</td>
<td>A standard double-precision 64-bit IEEE 754 floating point number. This matches the native JavaScript <code>Number</code> type.</td>
</tr>
<tr>
<td>String</td>
<td>A standard byte sequence.</td>
</tr>
<tr>
<td>Boolean</td>
<td>A true/false value. Useful for situations in which oracle service nodes must vote on an outcome (see "democracy" consensus mode)</td>
</tr>
</table>

As a consequence of this new return type feature, Moracle now supports __multiple consensus modes__ for accessing a variety of different internet API services. As Moracle provides a decentralized oracle service, multiple nodes can participate in the process of providing data to a blockchain app.

An API lookup to fetch data on a particular cryptocurrency transaction will always return consistent information, allowing full consensus among nodes. However, a request for volatile information such as an asset price does not allow for full agreement. Additionally, some APIs are *stateful*, meaning multiple nodes attempting to send a request will result in unexpected or unwanted outcomes. Moracle offers the following consensus options:
<table class="prettytable">
  <tr>
    <th>Option</th>
    <th>Compatible types</th>
    <th>Compatible options</th>
  </tr>
  <tr>
    <td>C - full consensus required, all nodes must produce the same outcome</td>
    <td>All</td>
    <td>T</td>
  </tr>
  <tr>
    <td>T - trusted responders only, only allow nodes with a key approved by the developer to contribute a response</td>
    <td>All</td>
    <td>S, C, F, M, O, D (all)</td>
  </tr>
  <tr>
    <td>F - first response wins, all nodes race to provide response, only the first response to the request shouldn't be ignored</td>
    <td>All</td>
    <td>S, T</td>
  </tr>
  <tr>
    <td>S - single request required, only one node should execute the request prior to storing the results, useful for stateful APIs. This is different from F in that it ensures only one request is sent.</td>
    <td>All</td>
    <td>T</td>
  </tr>
  <tr>
    <td>M - compute mean value of node results</td>
    <td>Number, Boolean</td>
    <td>T, O</td>
  </tr>
  <tr>
    <td>O - discard outliars from a Number result set, used with mean value option</td>
    <td>Number</td>
    <td>M, T</td>
  </tr>
  <tr>
    <td>D - democracy, the most popular value wins</td>
    <td>Number, String, Boolean</td>
    <td>T, O</td>
  </tr>

</table>

The options detailed above are combined to create a consensus mode. The *compatible options* column dictates which options can be used in the same mode. For example:
- FT - The first response is immediately taken as the correct result, the response must be signed by a key specified as trusted by the sidechain developer.
- MOT - The mean value from trusted nodes is returned after removing outliars (>2 SD from mean).
- D - The most popular value is returned.
 
This system allows developers building on the Lisk SDK simplicity and total flexibility in how their decentralized data providers work.

## Upcoming releases

### April 2021 - Moracle transaction library + service executor to enable oracle functionality + documentation

This release differs from previous Moracle prototypes in two significant ways. The largest change is that a unified GraphQL schema for all data sources is no longer used. This approach made for a novel developer experience, but was too complicated and led to repetitive code. Rather than using GraphQL, [JavaScript transaction implementations](https://lisk.io/documentation/lisk-sdk/guides/app-development/asset.html#applying-the-transaction-asset-logic) can use SQL syntax to read data. Additionally, a separate service which communicates with the Lisk node is used to execute requests made to data services rather than the Lisk node itself.

This library will be ready-to-use in your dApp. However, this version will require decentralized app operators to operate their own oracle resolver services or solicit volunteers to do so. These services can simply be hosted on the same cloud server that hosts the Lisk node.


### May 2021 - Moracle UI. Easily visualize and deploy oracle data sources for your decentralized app

Node operators can provide multiple services to multiple blockchains simultaneously. For example, a Bitcoin price service may be useful for multiple sidechains or one complex sidechain may use multiple Moracle services. 

The Moracle UI allows operators and developers to easily create, manage, configure, and destroy data services. 

<div class="captionedimg">
<img src="/img/2021-03-27-MoracleUI.png">
This dashboard allows developers and node operators to easily view and modify data sources for blockchain apps.
</div>


I've learned a lot since 2019 and am striving to provide the best developer experience possible. This project has suffered from some significant delays, but it's __still alive__ and will finally ready for developer use by May. 

I am looking forward to hearing your feedback over the next few months. Thanks to Lightcurve for making such an excellent SDK and the Lisk community for their interest in Moracle.
