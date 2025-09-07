---
title: Fedora Atomic Survival Guide
description: >-
  How to daily drive a Fedora Atomic distro
categories: [Linux]
tags: [linux]
pin: false
media_subpath: '/assets/img/posts/atomic'
---

## Overview
Recently, I've had a lot of people ask about my [Fedora Atomic](https://fedoraproject.org/atomic-desktops/) setup. Specifically, on my laptop I daily drive [Aurora](https://getaurora.dev/en), which is one of [Universal Blue's](https://universal-blue.org/) customized Fedora Atomic images. Their images add a lot of quality-of-life features that the default Fedora Atomic images lack, like [Homebrew](https://brew.sh/), Nvidia drivers, and more (depending on the image you choose). 

> "An immutable Linux distribution is an operating system (OS) that is read-only at its core. That means you can't easily modify the OS. This includes the file system, directories, applications, and even configurations. Even as an administrator, you can't make any modifications to the distribution."
> 
> This offers security, maintenance, and reliability benefits.
> 
> Quoted from [HowToGeek](https://www.howtogeek.com/what-is-an-immutable-linux-distro/)
{: .prompt-info }

> Did you know? SteamOS on the Steam Deck uses an immutable version of Arch Linux. If you're looking for a similar experience for gaming on Linux, check out Universal Blue's [Bazzite](https://bazzite.gg/).
{: .prompt-info }

I enjoy Fedora Atomic for its combination of stability and ease of use - it's slightly more difficult to use than regular Linux, but much more resistant to a degraded installation over time. Fedora also happens to be one of the officially supported Linux distros for my Framework laptop, so I enjoy a smooth driver experience. I've been running the same installation since January 2025 with no major issues and with seamless automatic updates to my system (another advantage of atomic/immutable distros).

With that said, there is a slight learning curve that comes with running an immutable system. You can't install packages with your package manager (`dnf` on Fedora) like a regular Linux distro. A large part of the stability comes from the fact that your system packages are immutable. I am writing this guide to help newcomers understand how to overcome this limitation.

## Tools of the Trade
There are 6 primary methods I use to install software on Fedora Atomic. I am going to assume you are driving a Universal Blue image with Homebrew and Distrobox already installed, but this could work on any image so long as you install those things first.

Primary methods of installation (loosely in order of preference):
- Homebrew
- Flatpak
- AppImage
- Distrobox
- Virtual Machines
- Layering with `rpm-ostree`

I will cover each of them in more detail below.

## Homebrew
[Homebrew](https://brew.sh/), or `brew`, essentially serves as your system's primary package manager. You might be familiar with it if you've ever used MacOS. It also happens to work on Linux (mostly, there are a few MacOS-only packages), so you'll benefit from the same expansive selection of software that MacOS does. When I'm trying to install a CLI tool, the vast majority of the time I can find it here.

> Useful commands:
> ```shell
> brew search <package-name>
> brew info <package-name>
> brew install <package-name>
> ```
{: .prompt-tip }

> Important things I get here:
> - neovim
> - pyenv
> - rust, ruby, go, etc.
> - several improved CLI navigation/viewing tools like moar, eza, stow, and zoxide
{: .prompt-info }

## Flatpak
Flatpaks from [Flathub](https://flathub.org/) are my second-most frequently used installation method. Most GUI apps you'll care about are available here. Flatpak is containerized, distribution-agnostic package format. Keep in mind that not all Flatpaks are official ("Verified") sources of installing the app, so you'll have to weigh the pros/cons of that yourself. You'll also want to be aware that because Flatpaks are containerized applications, in some cases their functionality can be limited compared to their non-Flatpak counterparts. For example, if you use a password manager like 1Password, the 1Password browser extension installed in the Flatpak version of a web browser cannot integrate with the host 1Password application (due to the sandbox).

> Useful commands:
> ```shell
> flatpak search <package-name> # or use the flathub link from above
> flatpak install <package-name>
> ```
{: .prompt-tip }

> Important things I get here:
> - Ghidra
> - Podman Desktop (alternative to Docker Desktop)
> - Discord
> - Flatseal (permissions manager for Flatpaks)
> - Obsidian
> - Bottles (Wine wrapper - runs Windows applications)
> - BoxBuddy (manager for Distrobox)
> - ONLYOFFICE
> - Zoom
> - etc.
{: .prompt-info }

## AppImage
Several applications will offer an official AppImage as the Linux installation method. Think of them as a worse version of Flatpak (because while they're supposed to be portable and distribution-agnostic, that isn't always the case). There's no package manager or CLI tool for this - you'll have to look on the application's website to see if they offer one.

> Important things I get here:
> - Logic 2
{: .prompt-info }

## Distrobox
Distrobox is a fantastic tool, even outside of immutable systems. I like to think of it as the WSL equivalent for Linux. Distrobox is a wrapper around Podman or Docker (your choice) that allows you to easily spin up containers that are automatically integrated with your home directory. It also allows you to "export" CLI and GUI apps to your host so you can run them as if they were installed on your host machine. This means that even if you have a piece of software that cannot be installed through any of the above three methods, you can likely install it in a distrobox and be just fine. I also like to keep a Kali distrobox with the default set of tools preinstalled for easy access (even better than a VM).

![Boxbuddy Manger](boxbuddy.png)
_Boxbuddy distrobox manager app_

> Useful commands:
> ```shell
> distrobox create --image <image-name> --name <distrobox-name>
> distrobox enter <distrobox-name>
> ```
{: .prompt-tip }

> Important things I get here:
> - Burp Suite (Kali distrobox)
{: .prompt-info }

## Virtual Machines
If for some reason one of the above methods does not work, you also have the option of a Virtual Machine. Your best hypervisor option on Linux is KVM with the Virt Manager frontend. This is actually even better than the Linux versions of VirtualBox and VMWare or hypervisors on Windows because KVM is built-in to the Linux kernel, which results in huge performance gains.

![My VMs](vms.png)
_My VMs in Virt Manager_

> Important things I get here:
> - Kali VM
{: .prompt-info }

## Layering
Sometimes, for a variety of reasons, none of the above methods will work. In this case, your last resort is to layer the package onto your image with `rpm-ostree`. You can layer any package normally installable with `dnf` using this method. The reason it's not recommended unless necessary is because every time you update your system, it will have to re-layer this package on top of the base image, which adds time to the update process. Plus, it technically breaks the "immutable" system principle, so the more packages you layer, the greater the chance of an unstable system. Whenever you layer something new, you'll have to reboot your system to load the updated image.

> Useful commands:
> ```shell
> rpm-ostree status
> rpm-ostree install <package-name>
> ```
{: .prompt-tip }

> Important things I get here:
> - 1Password
> - Firefox (I personally don't like the Flatpak version)
> - kitty (my preferred terminal emulator)
> - etc.
{: .prompt-info }

## Resources
Here are some other resources that might be helpful for learning how to use an immutable distro:
- [What is an immutable Linux distro? HowToGeek](https://www.howtogeek.com/what-is-an-immutable-linux-distro/)
- [Atomic Desktops - Chris Titus Tech](https://youtu.be/1vSFR8bi0YQ?si=By2ViztAGgQUz-gK)