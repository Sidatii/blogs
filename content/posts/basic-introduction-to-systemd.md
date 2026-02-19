---
title: Linux from scratch - Introduction to Systemd [Concepts]
category: Linux
tags: OS - system - Engineering - biginner
excerpt: This blog serves as an introduction to engineers interested in understading how linux system operates under the hood.
# published_at: 2026-02-19
---

# Systemd

## Introduction

As engineers, our core motivation must always be understanding how things work under the hood. However, and in the AI era, this becomes more and more costly to maintain given expectations about productivity and performance. This imposes a reality of just wanting to get the job done regardless of understanding what we do. As someone who feels the same about the world we live in, I always want to remind myself to use the tools but never outsource problem solving.

In this series of Linux from scratch, I am documenting my learnings about things that matter the most to me in Linux architecture. And as a person who cherishes Linux and its ecosystem, I am taking you with me throughout this journey.

## Definition

Systemd is a system and service manager for Linux operating systems, designed to provide a standard process for controlling the startup, shutdown, and management of system services. It replaces the traditional init system and offers features like parallel service startup, dependency management, and socket-based activation.

Systemd is an init system that initializes user space components after the Linux kernel has booted. It is responsible for starting and managing system services, handling system resources, and providing a unified interface for service management.

It is the most important process on Linux. It has a process ID of 1.

## Units

Systemd uses the concept of "units" to represent resources that it manages. Units can be services, sockets, devices, mount points, and more. Each unit has a configuration file that defines its behavior and properties.

## Configuration Files

Systemd configuration files are typically located in `/etc/systemd/system/` for user-defined units and `/lib/systemd/system/` for system-provided units. Each unit file has a `.service`, `.socket`, `.mount`, or other appropriate suffix, depending on the type of unit.

## Systemd Unit Directories

Systemd unit files are organized into several directories:

- `/etc/systemd/system/`: User-defined unit files. These override the system-provided unit files.

- `/run/systemd/system/`: Runtime unit files, which are created at boot time and can be modified during runtime.

- `/lib/systemd/system/`: System-provided unit files. These are installed by packages and should not be modified directly.

- `/usr/lib/systemd/system/`: Similar to `/lib/systemd/system/`, but used in some distributions.

- `/etc/systemd/user/`: User-specific unit files for user services.

- `/run/user/<UID>/systemd/`: Runtime user-specific unit files.

## Unit directories priority

When systemd looks for a unit file, it checks the directories in the following order:

1. `/etc/systemd/system/`

2. `/run/systemd/system/`

3. `/lib/systemd/system/`

## Example Unit File

```ini
[Unit]
Description=Example Service
Wants=network-online.target
After=network.target

[Service]
Type=simple
Environment=EXAMPLE_ENV_VAR=value
ExecStart=/usr/bin/example-service
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## Edit Unit File

To edit a systemd unit file, you can use the `systemctl edit` command, which creates an override file in `/etc/systemd/system/`. This allows you to modify the unit without changing the original file.

```bash
systemctl edit example.service
```

This generates an override file in `/etc/systemd/system/example.service.d/override.conf`, where you can add your custom configurations.

>[!NOTE] This triggers a partial update of the unit file, meaning only the changes you make will be applied, and the rest of the unit file remains unchanged.

If you want to apply a full update, you can use the `systemctl edit --full` command:

```bash
systemctl edit --full example.service
```

This will open the entire unit file for editing, allowing you to modify any part of it.

## Reload Systemd Configuration

After making changes to unit files, you need to reload the systemd configuration to apply the changes:

```bash
systemctl daemon-reload
```


## Summary

Systemd is a powerful and flexible system and service manager for Linux, providing a unified way to manage system resources and services. It uses units to represent various system components, and its configuration files are organized into specific directories. Systemd allows for easy management of services, including starting, stopping, and reloading configurations, making it an essential tool for system administrators.


> [!IMPORTANT] If you like the journey, you can *Subscribe* to my newsletter to get notified whenever new blogs are published!
