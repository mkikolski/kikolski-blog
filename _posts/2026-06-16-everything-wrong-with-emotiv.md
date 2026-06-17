---
layout: post
title: "Everything wrong with Emotiv - Extracting paywalled data"
subtitle: "For legal reasons, this is not a guide"
date: 2026-06-16 19:00:00 +0200
tags: [eeg, corporate greed, reverse engineering, coding]
excerpt: "Emotiv is a company offering various consumer-grade EEG devices with BCI capabilities..."
---

Emotiv is a company offering various consumer-grade EEG devices with BCI capabilities, and I had a dubious pleasure of working with their Emotiv EPOC X device for the last few months. Disregarding the fact that the intended API connection and data retrieval mechanism is unnecessarily complicated, obtaining some precomputed metrics is not that bad (despite the fact that the API is terribly documented). The problems arise when you try to get raw EEG data from your legally purchased device - it's impossible without paid license (around $225 for yet limited features, the full feature set requires individual quota). Unfortunately for Emotiv, I'm **1)** allergic to corporate greed **2)** quite experienced with building video game cheats **3)** determined to get to the data. The whole process wouldn't need to take place if the company fulfilled our order and supplied us with the USB dongle for connecting the EPOC X to the PC, because there are already existing apps for retrieving the data from the dongle.

## How to cheat in video games

For this we will need to dive into some computer science. Every program uses RAM to store data used in the runtime. For video games these are often health values, ammunition counts, player and enemies positions and other useful information. But there's a problem - we don't know where in the memory these values reside. Fortunately, there are tools that allow us to read those values and find the memory addresses - the most common of them is Cheat Engine. The Cheat Engine allows us to scan selected process memory for certain values, which can be a good start. When working with FPS games, the easiest value to search for is the health value. We just attach to the process of the game, enter a training match and search for initial health value. For screenshots and scanning in this post, I'll be using [AssaultCube](https://assault.cubers.net/) - an opensource game often used for learning reverse engineering. The game looks like this:

![AssaultCube Interface](/assets/images/assault_cube.png)

After opening the Cheat Engine, we can attach to the AC process and scan for current HP value:

![Cheat Engine Window](/assets/images/cheat_engine_1.png)

The scan has yielded a few hundred results alongside their memory addresses:

![Scan results](/assets/images/cheat_engine_2.png)

Then, I've taken damage in game and rescanned with the new value:

![Modified HP value](/assets/images/assault_cube_hp.png)
![Modified scan results](/assets/images/cheat_engine_3.png)

Now we can see that there are 2 addresses left and further scanning could be pointless - one of them might be some temporary value or just a value copied for draw calls while the second one is the HP value in the Player struct that actually interests us. We can use the Cheat Engine builtin tools to see which addresses write to this health address. This is particularly interesting because we might see the whole Assembler based path of writes that will reveal the base of the struct.

![Write scan](/assets/images/cheat_engine_4.png)

Here, in particular, we see that one of our found addresses is used for subtraction and is referenced as `EBX+0x04` - this suggests that register `EBX` holds the base address of the player struct and `0x04` is a pointer from base struct to the HP value. But...

...finding the address is just the beginning of the search as the executables are loaded at a random position in the memory every time the app launches, so we need to find a way to preserve our findings between the app launches. Even though the base address is always different, the offset between the game executable and certain values is usually the same, so if we can find the offsets from `ac_client.exe` to our player struct base, we will be able to access it anytime. Once again, Cheat Engine has us covered. My way of finding the proper offsets is performing a pointer scan. We can set the maximum length of the chain of the pointers that will lead from the executable to our address and the max size of single pointer. For the scan, I opted to limit the depth to 3 pointers and let it scan for a few seconds:

![Pointer scan](/assets/images/cheat_engine_5.png)

As you see, we've got quite a few possible paths, so now comes the boring part - we need to save the scan results, close the game, open it once again, find the player struct base address once again and do another pointer scan - this time referencing the previous scan's results. This will eliminate the redundant options and single out the best candidates. Sometimes, you will need to perform this action a few times to get left with a single path. 

When we're left with a single path, we can finally build an external program that reads or writes to this memory.

> [!NOTE]
> Of course, when hacking games, almost every game will have some anti-cheating measures implemented, so we won't be able to freely modify our health or speed but for Emotiv apps, we will use only reading anyway.

When making video game cheats, I used C++ to develop them as it is performant, is relatively close to the OS internals and can be compiled, but this time we will use Python as I don't need the performance and I don't concern myself with an anticheat. Python has wrappers for process memory management macros available in the [`pymem` package](https://pypi.org/project/Pymem/). The basic read code looks like this:

```py
from pymem import Pymem
from pymem.ptypes import RemotePointer

base_offset = 0xF528
offsets = [0xB20, 0xB8]
hp_offset = 0x4

pm = Pymem("ac_client.exe")

def get_pointer_address(base: int, offsets: list[int]) -> int:
    remote_pointer = RemotePointer(pm.process_handle, base)
    for offset in offsets:
        if offset != offsets[-1]:
            remote_pointer = RemotePointer(pm.process_handle, remote_pointer.value + offset)
        else:
            return remote_pointer.value + offset

hp = pm.read_int(get_pointer_address(pm.base_address + base_offset, offsets=offsets + [hp_offset]))
```

Where the base offset and offsets in a list are results from the pointer scans and the HP offset is the `0x4` we've noticed earlier that was added to `EBX` containing base address when writing the health value.

This is not a game hacking tutorial so we'll end this part here, but maybe I'll write some in depth explanation of these techniques some time later this year. 

## How does it apply to my Emotiv problem?

After several attempts at establishing Bluetooth connection with my EPOC X myself and cracking the received data, I gave up because I needed a solution fast. I downloaded Emotiv Client and Emotiv Pro apps and began trying to intercept the data. I've successfully found the addresses of battery level and the connection quality using previously mentioned Cheat Engine techniques and Emotiv's simulated device but they didn't lead me to an interesting addresses and structures. This checked out with the API docs which turned out useful this one time - different types of values are sent in different streams and I was back to the drawing board.

Then, according to the API reference, I tried searching for the float values of no known initial value (in previous part we've been only searching for 4 byte, exact values) and then alternating between values close to 0 and far from 0 while simultaneously changing the signal preset in the emulated device. This gave me some candidates and after a couple dozen minutes of manually checking the remaining addresses, I've found an array of 14 float values, each 8 bytes apart from the previous one - this suggested that the values are sent as float64s. 

Similarly to the Assault Cube, I've proceeded to look for some base address for that structure and after finding one hiding in one of the registers, I followed up with a pointer scan. This took a substantial amount of time and dirty martinis to accomplish and I was left with a clear offset list: `[0xFC0, 0x70, 0x70, 0x380]`. I'm not disclosing the app version nor the base offset, as you probably could find them yourselves at this point and I'm afraid that Emotiv might not like this approach - I will post the whole source code with some improvements and more advanced hacking techniques as an update in the future, when I switch from using this bypass to some more elegant solution.

## What now?

When we access the data in our Python script, we can do virtually anything - I decided to turn the Python script into a simple WebSocket server streaming this data into a simple frontend that allows saving the files.

However, this is far from over, even as a temporary solution:
- I should add a module that will scan for contact and EEG quality
- In case of the app updates, I should implement some pattern based offset scanning instead of doing it manually
- Even the Emotiv Pro Lite version has limits on logins to the app and we need to ditch it as soon as possible

If you wish, use this post as a reference in accessing YOUR data that might be paywalled. Keep tuned for updates, as I plan to update this solution sometime in the next month for the sake of researchers dealing with this shitty company.
