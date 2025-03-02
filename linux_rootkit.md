linux_rootkit
=============
- Category: Learning
- Tags: 
- Created: 2025-03-01T10:30:51-08:00

- rlkm Bias trigger
- Rootkit made to do things you can't really do normally
- Extension of trigger daemon
	- Triggerd - userspace command that senses things
- Ex.
	- Someone's about to do a brute force
	- Detect hydra or whatnot
	- Sensor -> triggerd -> rootkit
- Rootkit
	- Linux kernel module
		- Find online on how to do this - not malicious
		- Install like normal
	- LKM = Linux Kernel Module
- We wanted to intercept system calls
	- Pretty much everything relies on syscalls
	- If we intercept syscalls, we can do whatever we want
		- But how?
			- Kernel exposes ftrace
				- Meant for debugging - testing times, inputs, etc.
				- Allows you to install a "hook"
					- If you give it the address of a func. in kernel, whenever it's called, it'll check if there's a hook for that address
				- Look for memory address of the syscalls, then hook into them
	- How do we find syscall addresses?
		- Look for kallsyms_lookup_name. Not exposed in later versions to look
		- Can brute force it - look up symbol at each address
			- Not reliable
		- Use kprobe (another debugger) to look up

### Questions

- Is this easily detected?
	- Can hide self in `/proc`, other ways as well. You can go pretty deep.
- Are there any examples of this online?
	- Yup. Look for open source Linux rootkits.
