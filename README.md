<p align="center">
  <img width="180" alt="tux" src="https://github.com/user-attachments/assets/ee47f060-46b6-4aee-9d10-ea740d5b1751" />
</p>


## Notes for Linux Release 0.01

### 0. Contents of this directory

- `linux-0.01.tar.Z` – sources to the kernel  
- `bash.Z` – compressed bash binary if you want to test it  
- `update.Z` – compressed update binary  
- `RELNOTES-0.01` – this file  

### 1. Short intro

This is a free minix-like kernel for i386(+) based AT-machines. Full source is included, and this source has been used to produce a running kernel on two different machines. Currently there are no kernel binaries for public viewing, as they have to be recompiled for different machines. You need to compile it with gcc (I use 1.40, don't know if 1.37.1 will handle all `__asm__` directives), after having changed the relevant configuration file(s).  

As the version number (0.01) suggests this is not a mature product. Currently only a subset of AT-hardware is supported (hard-disk, screen, keyboard and serial lines), and some of the system calls are not yet fully implemented (notably `mount`/`umount` aren't even implemented). See comments or readme's in the code.  

This version is also meant mostly for reading – i.e., if you are interested in how the system looks like currently. It will compile and produce a working kernel, and though I will help in any way I can to get it working on your machine (mail me), it isn't really supported. Changes are frequent, and the first "production" version will probably differ wildly from this pre-alpha release.  

#### Hardware needed for running Linux:

- 386 AT  
- VGA/EGA screen  
- AT-type harddisk controller (IDE is fine)  
- Finnish keyboard (oh, you can use a US keyboard, but not without some practice :-)  

The Finnish keyboard is hard-wired, and as I don't have a US one I cannot change it without major problems. See `kernel/keyboard.s` for details. If anybody is willing to make an even partial port, I'd be grateful. Shouldn't be too hard, as it's table-driven (it's assembler though, so …).  

Although Linux is a complete kernel, and uses no code from Minix or other sources, almost none of the support routines have yet been coded. Thus you currently need Minix to bootstrap the system. It might be possible to use the free Minix demo-disk to make a filesystem and run Linux without having Minix, but I don't know…

### 2. Copyrights etc

This kernel is (C) 1991 Linus Torvalds, but all or part of it may be redistributed provided you do the following:

- Full source must be available (and free), if not with the distribution then at least on asking for it.  
- Copyright notices must be intact. (In fact, if you distribute only parts of it you may have to add copyrights, as there aren't (C)'s in all files.) Small partial excerpts may be copied without bothering with copyrights.  
- You may not distribute this for a fee, not even "handling" costs.  

Mail me at torvalds@kruuna.helsinki.fi if you have any questions.  

Sadly, a kernel by itself gets you nowhere. To get a working system you need a shell, compilers, a library, etc. These are separate parts and may be under a stricter (or even looser) copyright. Most of the tools used with Linux are GNU software and are under the GNU copyleft. These tools aren't in the distribution – ask me (or GNU) for more info.

### 3. Short technical overview of the kernel

The Linux kernel has been made under Minix, and it was my original idea to make it binary compatible with Minix. That was dropped, as the differences got bigger, but the system still resembles Minix a great deal. Some of the key points are:

- **Efficient use of the 386 chip:** Minix was written on a 8088, and later ported to other machines – Linux takes full advantage of the 386.  
- **No message passing:** System calls are just that – calls. This avoids some problems with message queues.  
- **Multithreaded FS:** Makes the filesystem more complicated but allows running several processes concurrently without the performance hit induced by Minix.  
- **Minimal task switching:** Task switching only when really needed.  
- **Interrupts aren't hidden:** Handled mainly by machine code; device drivers are mostly interrupt routines.  
- **No distinction between kernel/fs/mm:** All linked into the same heap of code.  

Guiding principle: **get it working fast.** The kernel is simple yet powerful enough to run most Unix software.  

- Just one data structure for tasks.  
- Simple memory management using both paging and segmentation capabilities of the i386.  

### 4. The "kernel proper"

All routines handling tasks are in the subdirectory `kernel`. Includes `fork`, `exit`, scheduling, minor system calls like `getpid`, handlers for exceptions and traps (except page faults), and low-level device drivers (`get_hd_block`, `tty_write`, etc.).  

Currently all faults lead to an exit with error code 11 (Segmentation fault), and the system seems relatively stable (`crashme` hasn't – yet).

### 5. Memory management

Simplest part, handles page faults as they happen. Memory is managed using **paging and segmentation**:

- 386 VM-space (4GB) divided into segments (currently 64 segments of 64MB each)  
- Kernel memory segment maps all physical memory  
- Tasks get one segment each; paging fills the segment and handles duplicates (fork) and copy-on-write  

### 6. The file system

- Linux FS is the same as in Minix (for cross-compiling and mounting Linux partitions from Minix)  
- Minix-1.6.16 has a new FS, Linux won't understand it  
- Linux FS is multithreaded; handles deadlocks/race conditions by double-checking allocations  
- Kernel/fs/mm actions are atomic if you don't call `sleep`

### 7. Apologies :-)

This isn't yet the "mother of all operating systems". It's a source release for those interested in seeing what Linux looks like, and it's not fully supported. Questions, suggestions, and bug reports are encouraged via email.  

### 8. Getting it working

- Compile hardware dependencies into the system (edit `include/linux/config.h`)  
- Uncomment the right `equ` in `boot/boot.s` for your A-floppy  
- Run `make` → produces `Image` → copy to floppy (`cp Image /dev/PS0`)  

Without programs to run, the kernel cannot do anything. You need `update` and `bash` binaries in `/bin` (bash must be `/bin/sh`).  

---

<img width="200" alt="linus" src="https://github.com/user-attachments/assets/ecf7c9bb-d675-4764-b956-bdd802406396" />

### 9. Historical note: Linus' first email.

> **From:** torvalds@klaava.Helsinki.FI (Linus Benedict Torvalds)  
> **Newsgroups:** comp.os.minix  
> **Subject:** What would you like to see most in minix?  
> **Summary:** small poll for my new operating system  
> **Message-ID:** <1991Aug25.205708.9541@klaava.Helsinki.FI>  
> **Date:** 25 Aug 91 20:57:08 GMT  
> **Organization:** University of Helsinki  
>   
> Hello everybody out there using minix – I’m doing a (free) operating system
> (just a hobby, won’t be big and professional like gnu) for 386(486) AT clones.
> This has been brewing since april, and is starting to get ready. I’d like any
> feedback onthings people like/dislike in minix, as my OS resembles it somewhat
> (same physical layout of the file-system (due to practical reasons) among other things).
>   
> I’ve currently ported bash(1.08) and gcc(1.40), and things seem to work.  
>   
> This implies that I’ll get something practical within a few months, and I’d  
> like to know what features most people would want. Any suggestions are welcome,  
> but I won’t promise I’ll implement them 🙂  
> Linus (torvalds@kruuna.helsinki.fi)
>
> PS. Yes – it’s free of any minix code, and it has a multi-threaded fs.
> 
> It is NOT protable (uses 386 task switching etc), and it probably never will
> support anything other than AT-harddisks, as that’s all I have 😦


<img width="400" alt="linus-young" src="https://github.com/user-attachments/assets/b44fab67-9270-48a6-b602-deb1e50e3bb7" />

**Linus Torvalds** – torvalds@kruuna.helsinki.fi  
Petersgatan 2 A 2  
00140 Helsingfors 14, FINLAND

Thanks Linus 🐧
