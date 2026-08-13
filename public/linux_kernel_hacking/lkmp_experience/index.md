# My Linux Kernel Mentorship Program Experience: A Fantastic Trip


My experience in the [Linux kernel Spring Unpaid 2026](https://mentorship.lfx.linuxfoundation.org/project/53378ec5-48d7-4c49-a01f-8cbd3948db3d), and the beginning of my journey as a Linux kernel developer.

<!--more-->

## How it started

I have always been passionate about electronics, and especially about that thin boundary where a physical object meets the software that drives it. Throughout my electronics studies at university, what fascinated me most was understanding, in depth, how a specific behavior is built into digital hardware, and above all how that hardware is designed from the very start to be configured and steered from software. That curiosity pushed me to dig deeper. After a detour through the firmware world, I came across a YouTube video in which Linus Torvalds put into words exactly what I had been feeling: a deep interest in controlling hardware through software, deciding its behavior down to the smallest detail. That is where my passion for the Linux kernel took root. Here was a real operating system, born as an open source project and at the same time a huge collection of device drivers: the most concrete form of what I wanted to do. Up until the experience I am about to describe, I had invested everything in hardware design and firmware development, so when the time came to choose a direction for my career, my ideas were already fairly clear.

I wanted to take the next step: to control and configure everything I placed on a printed circuit board, and everything I found on a third-party system-on-chip or system-on-module, in order to get the most out of that hardware. What I also realized is that the initial learning curve of the Linux kernel is very, very steep. It is nothing like working with a "simple" operating system such as FreeRTOS, widely used on microcontrollers. The Linux kernel is far more complex, and each subsystem has its own way of behaving and its own way of modeling the hardware underneath it.

Still hungry to learn more, I kept searching online, and that is how I came across the blog of [Javier Carrasco](https://hackerbikepacker.com/lkmp), where he describes his own experience in the Linux Kernel Mentorship Program, first as a mentee and later as a mentor.

## Applying to the program

At that point I had found what I was looking for: a practical, guided way into the Linux kernel, with the support of kernel maintainers themselves. A path to my first patches, backed by two experts in the field. The minimum requirement was five patches.

So I applied. The application asked for two things: the course [A Beginner's Guide to Linux Kernel Development (LFD103)](https://training.linuxfoundation.org/training/a-beginners-guide-to-linux-kernel-development-lfd103/), and a few tasks on kernel development and on the debugging techniques to use when things go wrong, mostly topics the course already covers.

I admit I underestimated that course. It covers a whole set of skills that, I can say today, make up a large part of a Linux kernel developer's job. The job is not just writing code: it is writing it well, describing it well, and reshaping it around the feedback of reviewers and of the maintainer of the subsystem you are contributing to.

So if you are thinking about applying, my advice is simple: go through the pre-course carefully and with curiosity. Followed properly, it gives you everything you need to start contributing to the Linux kernel in a clean and organized way from day one.

## Getting accepted

To my surprise, on March 2, 2026 I received a message from Shuah Khan: I had been admitted to the Linux Kernel Mentorship Program (LKMP), Spring 2026. The program comes in more than one form, paid or unpaid, three months or six, and I had been accepted into the unpaid, six-month, part-time track. That was the only one I could realistically take on, since I was going to follow the program alongside my full-time job.

The recommended commitment for this track was 20 hours a week, and starting from scratch as I did, I used every single one of them, especially during the first three months. More than once my evenings turned into a second working day. But you do that and more when you are passionate about something.

## The first meeting

The first online meeting took place a few days after the admission. The maintainer Shuah Khan and the developer Brigham Campbell introduced themselves as the mentors of this LKMP Spring 2026. During this first meeting, a few important points immediately reset my expectations:

- The Linux Kernel Mentorship Program is not like a university course. You are not led through a predefined study path. I would describe it more as an experience where, if you commit and ask the right questions, the mentors will give you excellent hints and advice, pointers within the whole space of possible solutions, so you don't waste time on dead ends and get straight to the point. But there is an initial step: you have to work hard to reach the point where you can ask the right questions. This will become clearer later...

- The other mentees are a resource to make the most of. Collaborating, asking, comparing notes: i think it's essential, especially at the beginning, when you can easily feel lost in front of the immense range of possibilities you face when you first open the Linux kernel source.

## Choosing a subsystem

The first big task, or rather the goal of the first months, was choosing a kernel subsystem to start working on. That feeling of being lost hit me: a huge number of options in front of me, none of them meaning much to me yet, because I did not know the kernel well enough.

One direction, though, was clear. My interest in controlling electronics, now seen from the kernel side, pointed naturally to the Industrial I/O (IIO) subsystem, which I had discovered through Javier's articles. Its maintainer, Jonathan Cameron, describes it as "the IoT subsystem of the Linux kernel". I was also intrigued by the RISC-V architecture, which I had studied from a digital design perspective during my master's degree and could now look at from a completely different angle. I soon realized, however, that RISC-V had a much higher entry barrier: the work waiting there assumed a level of familiarity with the kernel that I simply did not have yet, while IIO offered patches I could actually start from. So I set RISC-V aside and focused entirely on IIO.

I had figured out the "what", but I had no idea about the "how". Where do you find bugs? And once you find one, how do you turn it into a patch?

So I told Shuah and Brigham that I was stuck, and they reassured me: I was exactly where I was supposed to be. They also gave me something practical to hold on to, and that is how I discovered the so-called low hanging fruit: small, self-contained patches, such as documentation fixes.

## My first patch: even a comment counts

Following the mentors hint to always work on the `mainline` or `linux-next` branch, i ran:

```bash
make htmldocs 2>&1 | grep WARNING
```

That command led me to my first patch opportunity, in the USB Type-C subsystem (*include/linux/usb/typec_altmode.h*): two fields of `struct typec_altmode`, `priority` and `mode_selection`, were missing their kernel-doc entries, causing warnings when building the documentation.

A documentation patch may sound trivial, but it made me realize right away that even a "simple" comment can be a valuable contribution. To write those few lines i had to use `cscope` to trace where the two fields were actually used and deduce their real meaning. Along the way i discovered the concept of USB-C "alternate modes" and how the `priority` field drives the automatic alternate mode selection process. This patch was also my first complete round of the kernel submission process: configuring my git identity,`git commit -s`, `git format-patch`, the `checkpatch.pl` script, `get_maintainer.pl` to find the right recipients, and finally `git send-email`. It's also when i settled on a workflow that kept all my subsequent work in order: one dedicated git branch per patch, named like *patch/subsystem_name/short_description*, so every piece of work stays isolated and easy to rework when reviews come in.



The commands that carried this first patch, from code navigation to the send:

```bash
# Index the tree for cscope (kernel mode), then browse definitions and callers
find . \( -name '*.c' -o -name '*.h' -o -name '*.S' \) > cscope.files
cscope -b -q -k                # build the index once
cscope -d                      # ask the code: who defines/uses this symbol?

# One dedicated branch per patch
git checkout -b patch/usb/typec_altmode/v1 origin/master

# The submission round executed from the Linux Kernel source dir
git commit -s                                              # -s adds Signed-off-by
git format-patch -1 -o ../patch_typec/         # generate the .patch
./scripts/checkpatch.pl --strict --codespell ../patch_typec/0001-*.patch
./scripts/get_maintainer.pl ../patch_typec/0001-*.patch    # who goes in --to and --cc
git send-email --dry-run --to=... --cc=... ../patch_typec/0001-*.patch   # simulate first, always
git send-email --to=... --cc=... ../patch_typec/0001-*.patch
```

## My second patch: static analysis and my first reviews

Encouraged by that first submission, i followed another piece of advice from the weekly office hours: static analysis is a great source of beginner-friendly patches. So i enabled every driver as a module and built the whole kernel treating warnings seriously:

```bash
make allmodconfig
make W=1 -j$(nproc) > build_log.txt 2>&1
grep "warning" build_log.txt
```

That's how I found my second patch: an unused variable warning in *arch/x86/events/intel/p4.c*. Reading a model-specific register with `rdmsr()` splits its 64 bits into two 32-bit variables, `low` and `high`, and here `high` was set but never used.

But the fix itself is not what I remember most about this patch: it was my first time dealing with reviewers. My first attempt was the naive one, printing the unused half with `pr_cont`, so that the variable would count as used and the warning would go away. The reviewers pointed out that the rest of the x86 code had already moved past that pattern entirely: `rdmsrq()` reads the full 64-bit register into a single variable, so there is no second half left to worry about. The final version switched to `rdmsrq()` and, in the same patch, replaced a magic `(1 << 7)` with the proper `MSR_IA32_MISC_ENABLE_EMON` macro.

Lesson learned: before changing a line of kernel code, look at how the same pattern is handled elsewhere, because there is often an established "standard" way to do it.

On the practical side, i learned how to produce a v2 of a patch, reworking the commit with `git commit --amend` and regenerating it with `git format-patch`. And it started to dawn on me that submitting a patch is only half of the work: collaborating with the community is the other half.

This is the v2 cycle i learned here and reused for every review round afterwards:

```bash
# v2 gets its own branch: v1 stays intact, nothing is ever destroyed
git checkout -b patch/x86/p4/v2 patch/x86/p4/v1
git commit --amend                                  # rework code and commit message
git format-patch -1 -v2 -o ../patch_p4/ # -v2 puts [PATCH v2] in the subject
# then write the changelog below the --- line of the .patch (reviewers only, git am ignores it)
git send-email  --to=... --cc=... ../patch_p4/v2-*.patch
```

## Landing in IIO: my first real bug fix

At this point i had to commit to a subsystem for the rest of the mentorship, and given my electronics background, the IIO seemed to be  the most interesting. Sensors, ADCs and especially light sensors were home turf for me and RGB color sensors intrigued me the most.

The office hours gave me a method here too: read the subsystem's recent history to spot the "hot" topics being worked on. So i started digging with:

```bash
git log --oneline --no-merges drivers/iio/
```

Combining what i found there with Javier's articles, i ended up studying one of the light sensor drivers, *veml6070.c*, very closely. And there it was: a function that computed the sensor value correctly and then returned 0 instead of the computed result, so userspace always read 0. I had just done my first "code inspection", and what i had found was a real bug.

This little patch taught me a surprising amount:

- `git blame`, to trace which commit introduced the bug, which is what you need for the `Fixes:` tag;
- that a patch with a `Fixes:` tag gets backported to the stable trees: some time later i received the mail from [Greg Kroah-Hartman](https://en.wikipedia.org/wiki/Greg_Kroah-Hartman) confirming the patch had been merged into the right list of stable kernels;
- what a good bug-fix commit message looks like. Shuah taught me to always state how i found the bug, whether i tested it on real hardware, and how. In this case i didn't have the hardware, so i said so explicitly; the fix was obvious enough that the maintainer merged it anyway.

The bug-hunting commands behind this patch:

```bash
# Before touching anything: is anyone already working on this file? (check ALL branches)
git log --oneline --all -- drivers/iio/light/veml6070.c | head -20
# ...and search Patchwork and lore.kernel.org for pending patches too

# Find the commit that introduced the buggy line
git blame drivers/iio/light/veml6070.c | grep 'return 0'
# Generate the ready-to-paste Fixes: line from that commit
git log -1 --pretty=reference <sha>
```

## cm3323: my first series, and testing without the hardware

To learn the typical structure of an IIO driver i picked a fairly complete one: the CM3323 color sensor driver. While reviewing the code i found another bug, similar in spirit to the previous one: `data->reg_conf` was assigned the return value of `i2c_smbus_write_word_data()` (which is 0 on success) instead of the value actually written to the register, so the integration time always read back as 0.040000 no matter the configuration. Another bug fixing patch.



While looking at the changes in the latest patches to the subsystem, i noticed a pattern converting the probe error path to `dev_err_probe()`. This conversion taught me something conceptual about probing. When a probe fails to reach the hardware, it is not necessarily a hard failure: the driver may simply have started before its dependencies (controllers, GPIOs, and so on) were ready. `dev_err_probe()` handles exactly that case, deferring the probe instead of returning an immediate error, and it even simplified the code.

Since this change was more substantial than my previous ones, Shuah advised me to find a way to test it. The sensor turned out to be so old that it is out of production, so i couldn't buy one. That limitation became a lesson in itself: i discovered the **i2c-stub** technique, which let me simulate the I2C device and its registers on a Raspberry Pi 3B and actually test the patch. It was also my first **patch series**, with all the rules that come with submitting one.

This is the whole i2c-stub trick:

```bash
# Create the fake hardware: i2c-stub registers a new virtual I2C bus (on my
# machine it came up as i2c-11) with a simulated chip at address 0x10. Its
# "registers" live in RAM and answer reads/writes exactly like a real chip.
sudo modprobe i2c-stub chip_addr=0x10

# Two more modules are needed: i2c-dev exposes the buses to userspace tools
# like i2cset, and industrialio is the IIO core the cm3323 driver depends on.
sudo modprobe i2c-dev && sudo modprobe industrialio

# Pre-populate the configuration register BEFORE the driver probes: write the
# word 0x0030 into register 0x00 of chip 0x10 on bus 11. This plants a
# non-default integration time in the fake chip, exactly the value the buggy
# driver loses track of.
i2cset -y 11 0x10 0x00 0x0030 w

# Instantiate a cm3323 device at address 0x10 on the stub bus: this triggers
# the driver's probe(), which now reads and writes the fake chip's registers
# convinced it is talking to real hardware.
echo "cm3323 0x10" | sudo tee /sys/bus/i2c/devices/i2c-11/new_device

# Read through the driver's sysfs interface: with the bug,
# integration_time always reports 0.040000 no matter what is in the register;
# with the fix applied, it reports the value actually backed by the hardware.
cat /sys/bus/iio/devices/iio:device0/integration_time
```

## tcs3472: eight patches to close one TODO

The last and biggest piece of work was an 8-patch series on the TCS3472 RGBC light and color sensor driver. The series fixes a pre-existing bug where the chip was not powered down on probe failure, converts the driver to *devm-*based resource management, modernizes the locking with `guard(mutex)`, applies a few style cleanups, and finally implements wait time support exposed through the `sampling_frequency` sysfs attribute, closing a long-standing TODO in the driver.

What this series really taught me is the value of **preparatory patches**. They are not just cosmetic cleanups: they respect the principle of minimal change and structure the series so that the real functional change at the end is easier to understand, review and accept. Several rounds of feedback from the maintainer and the main reviewers forced me to reorganize the series multiple times, and every round made it more disciplined (and longer!).

A few individual lessons i took away:

- `guard(mutex)` removes the need for manual unlocks on every return path and eliminates `goto`-based cleanup, making the control flow much clearer;
- the `devm_*` APIs let the kernel core release resources automatically at cleanup time, and `devm_add_action_or_reset()` can register a custom power-down action, so the sensor is always left in a safe state even when probe fails;
- reading the datasheet in depth pays off. Implementing the wait time feature meant translating a real hardware capability (reducing power consumption between acquisitions) into a clean userspace interface, controlled by nothing more than the sampling frequency parameter.

This time i tested everything on real hardware: an Adafruit TCS3472 breakout that i bought and wired to my Raspberry Pi 3B on I2C-1. Manually creating the I2C device, binding the driver to it, verifying the binding, exercising the driver's event functionality: that hands-on part is where the driver really "clicked" for me. And honestly, this is the kind of kernel work i enjoy most: turning what a datasheet promises into a clean software interface.

The commands that shaped this series deserve a bit more space, because this is where the real testing happened. First, the research and development side:

```bash
# How does the subsystem already use the pattern i want to apply?
git grep -n -B2 -A5 "guard(mutex)" drivers/iio/light/
git log -S "guard(mutex)" --oneline -- drivers/iio/light/   # complete conversions to imitate

# Build only what you touched, then cross-compile the module for the Raspberry Pi
make drivers/iio/light/tcs3472.o W=1
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- M=drivers/iio/light/ modules
scp drivers/iio/light/tcs3472.ko aldo-rpi:/tmp/
```

Then the bring-up on the Raspberry Pi. This cycle ran dozens of times, once for every iteration of every patch (and for a series, each patch must be tested at its own):

```bash
# Is the sensor really on the bus?
sudo i2cdetect -y 1                  # expect 0x29 in the grid
sudo i2cget -y 1 0x29 0x92           # ID register: 0x44 = TCS34725

# Reload cycle: out with the old module, in with the freshly built one
echo 0x29 | sudo tee /sys/bus/i2c/devices/i2c-1/delete_device 2>/dev/null
sudo rmmod tcs3472 2>/dev/null
sudo insmod /tmp/tcs3472.ko
echo "tcs3472 0x29" | sudo tee /sys/bus/i2c/devices/i2c-1/new_device   # triggers probe()
sudo dmesg | tail -5                 # did the probe succeed?
```

Once the driver is bound, sysfs is the playground. The trick i used the most: comparing what the driver reports (`cat` goes through `read_raw()`) with what the chip actually contains (`i2cget` bypasses the driver), to check that the driver's cache is consistent with the hardware:

```bash
cd /sys/bus/iio/devices/iio:device0/

# Read the four channels through the driver
cat in_intensity_clear_raw in_intensity_red_raw in_intensity_green_raw in_intensity_blue_raw

# Exercise the new wait time feature via its userspace control
cat sampling_frequency
echo "10.0" | sudo tee sampling_frequency > /dev/null   # never `sudo echo >`: the redirect runs before sudo
cat sampling_frequency               # read back: the driver picks the closest supported value

# Cross-check driver vs hardware: does WTIME really change on the chip?
sudo i2cget -f -y 1 0x29 0x83        # WTIME register, read behind the driver's back
sudo i2cdump -f -y 1 0x29            # or dump the whole register space at once
```

And finally, since patch 3 touched the locking, a raw test: hammering the same attribute from concurrent readers and writers, then checking dmesg for trouble:

```bash
( for i in $(seq 1 500); do echo "5.0" | sudo tee sampling_frequency > /dev/null; done ) &
for i in $(seq 1 1000); do cat sampling_frequency > /dev/null; done
wait
sudo dmesg | grep -E 'BUG|WARNING|Oops|deadlock'   # silence is golden
```

When everything passed, the series went out. A series is not N single patches: it needs a cover letter and correct threading:

```bash
git format-patch -8 --cover-letter -o /tmp/series/
./scripts/checkpatch.pl --strict --codespell /tmp/series/*.patch 
git send-email --to=... --cc=... /tmp/series/*.patch      # git preserves order and threading
```

## What i take away

By the end of the program, my patches had landed in mainline. The complete set, with the full diffs, is available in [my GitHub fork](https://github.com/aldocontelk/linux-upstream/tree/github_master).

This mentorship turned my desire to contribute to the Linux kernel into something practical and effective. The discussions with the mentors and the other mentees showed me their workflows and improved mine, taught me how to collaborate with developers from all over the world, and showed me how much care goes into an open source project that has kept improving for years thanks only to emails.

It has been an invaluable experience, and i am grateful to have been part of it. If you are thinking about applying: do it, take the pre-course seriously, and get ready to work hard because this will help you make truly unexpected progress.


---

> Author: [Aldo Conte](https://github.com/aldocontelk)  
> URL: http://localhost:1313/printk-diaries/linux_kernel_hacking/lkmp_experience/  

