## Introduction  
Sumario ejecutivo: 

Problem: Online video gaming, particularly competitive gaming and esports, consistently has to confront the issue of cheating. This challenge has intensified with technological advancements in gaming, prompting a response from game developers in the form of sophisticated anti-cheat systems. 

During the last years, there has been an escalation on how cheats are working and upgrading:
     
1. Cheats that run in Usermode  

2. Cheats that run at kernel level

3. Kernel cheats with  BYOVD (with a custom vulerable driver)

4. Hypervisor-based cheats

5. DMA Cheats (Direct Memory access) 

6. Firmware-based attacks (SSD, RAM, GPU)


 

Solution: 

Make a kernel-based anticheat:

During the last years, there has been an escalation on how cheats are working and upgrading: 

- Cheats that run in Usermode  

- Cheats that run at kernel level 

- Kernel cheats with BYOVD (with a custom vulerable driver) 

- Hypervisor-based cheats 

- DMA Cheats (Direct Memory access)  

- Firmware-based attacks (SSD, RAM, GPU) 


## Architecture of a Kernel Anti-Cheat 

Early anti-cheat ran in userspace: a process monitored the game process for memory modifications, injected DLLs, or known cheat patterns.

This was effective against simplistic cheats but trivially bypassed: any process with sufficient privileges can inject code, manipulate memory, or hide itself from user-space monitoring.

The fundamental problem with usermode-only anti-cheat is the trust model. Any protection implemented entirely in usermode can be bypassed by anything running at a higher privilege level.

  

The practical implication of boot-time loading is also why Vanguard requires a system reboot to enable: the driver must be in place before the rest of the system initializes, which means it cannot be loaded after the fact without a restart. 


The three types of detections  

 
Kernel driver: Runs at ring 0. Registers callbacks, intercepts system calls, scans memory, enforces protections. This is the component that actually has the power to do anything meaningful. 

  

Usermode service: Runs as a Windows service, typically with SYSTEM privileges. Communicates with the kernel driver via IOCTLs. Handles network communication with backend servers, manages ban enforcement, collects and transmits telemetry. 

  

Game-injected DLL: Injected into (or loaded by) the game process. Performs usermode-side checks, communicates with the service, and serves as the endpoint for protections applied to the game process specifically. 
 
This 3 things are needed 

Vanguards kernel driver uses a whitelist of future dll loading 

The main difference between the different levels of privilege is the accessibility of memory and instructions. User mode (ring 3) applications are isolated from kernel mode (ring 0) applications, because kernel-mode determines how user-mode behaves, and usermode-mode applications therefore cannot access kernel memory. 
 

When do they load:

BattlEye and EAC Load when the game is launched 

Vanguard is loaded before most of the system has initialized (boot-start driver) 

Kernel-mode cheats could directly manipulate game memory without going through any API that a usermode anti-cheat could intercept. 
 

Their detections:

## Boot-time vs Runtime Driver Loading 

The distinction between boot-time and runtime driver loading is more significant than it might appear.

BattlEye and EAC load their kernel drivers when the game is launched. They are registered as demand-start drivers and loaded via a driver loader from the service when the game starts. They are unloaded when the game exits.


Vanguard loads `vgk.sys` at system boot. The driver is configured as a boot-start driver (`SERVICE_BOOT_START` in the registry), meaning the Windows kernel loads it before most of the system has initialized.

This gives Vanguard a critical advantage: it can observe every driver that loads after it. Any driver that loads after `vgk.sys` can be inspected before its code runs in a meaningful way.

Behavioral Detection and Telemetry (mouse, ML, IA) 

Anti-VM and Environment Checks 

Hardware Fingerprinting and Ban Enforcement 


- Behavioral Detection and Telemetry (mouse, ML, IA)  

- Anti-VM and Environment Checks  

- Hardware Fingerprinting and Ban Enforcement  
 

## References

- [s4dbrd - How-kernel-anti-cheats-work](https://s4dbrd.github.io/posts/how-kernel-anti-cheats-work/) 

- [Why anti-cheat software utilize kernel drivers](https://secret.club/2020/04/17/kernel-anticheats.html) 

- [Example EasyAntiCheat Exploit](https://aftermathlabs.net/blog/10/08/2021/) 




Valor: 

- [Wikipedia - Protection rings](https://en.wikipedia.org/wiki/Protection_ring) 

- [Wiki - What is a kernel driver](https://wiki.lckfb.com/en/linux-docs-tspi3-rk3576/linux-driver-Basics/kernel-driver-intro/kernel-module-drivers.html) 

- [Zach Vohries - CrowdStrike Analysis](https://x.com/Perpetualmaniac/status/1814376668095754753)

	 
