---
title: Deploy your Laravel app on your VPS
category: DevOps
tags: Laravel, Deploy, VPS, Security, CI/CD
excerpt: Your guide to deploying your first app on a private VPS - PHP, Nginx and Postgres as case study.
published_at: 2026-02-09
---

# Deploy your Laravel app on your VPS

In the era of cloud computing, we often find ourselves as software engineers drown in a world of abstraction where cloud providers provide us with their abstracted apis to interact with their machine. This creates detached reality especially for juniors where you find yourself no understanding what it is to deploy into a server. You may be good deploying to a cloud provider, and that is already great. However, you often do not know what actually happens under the hood. In this article, I am going to take you on an enjoyable journey of dealing with servers and understanding what actually happens under the hood.

Our environment for this article is a virtual private server (VPS) of a not so important provider. The machine runs ubuntu server 24.04. That is all you need to know for now. the machine has barely nothing besides linux coreutiles (you may need to install curl or ssh as per your need but nothing is needed at this stage.).

Our target app is a Laravel blade app using a Postgres database and an Nginx server.

