<pre>
<b>FrookSINATRA</b>
is a kernel driver and a Virtualbox(Hypervisor) patch to make possible hook of the LSTAR, even with patchguard (Up-to-date Windows 8.1 on July 2014) activated.


<b>DRIVER:</b>
On Load, the driver:
	1. saves the original lSTAR,
	2. asks the hypervisor (MSR Knocking) to save the original LSTAR too,
	3. writes the LSTAR to with the address of the hook function,
	4. ... Working here ...

On unload, the driver:
	6. restores the old LSTAR
	5. asks the hypervisor to forget the original LSTAR
	
	
<b>HYPERVISOR:</b>
	I changed the virtualbox (Dirty patch again, sorry) VT-X hypervisor HMVMXR0.cpp, to intercept read and write of MSR.
	When the kernel writes a magic value on a magic MSR, the LSTAR is stored.
	Each time patchguard is executed, it asks the MSR like this asm("rdmsr 0xC000005") <a href="http://pastebin.com/mGbFHkk5">Patchguard MSR Request</a>, the hypervisor intercepts the read, and give the original LSTAR value (legit one), even if it was hooked by the driver !
	
	This is working because when a sysenter/syscall is made, the LSTAR MSR isn't read via rdmsr instruction, but read by the CPU itself, and hypervisor isn't called. So the instruction flow is redirected to the real value of the LSTAR, the hook function, if LSTAR is hooked.
	
<b>NOTE:</b>
	The driver can be loaded with dsefix <a href="http://www.kernelmode.info/forum/viewtopic.php?f=11&t=3322">DSEFix</a>
	The driver can work on Windows 7 x64 without the hypervisor

<b>TODO:</b>
	* Make a version where all the thing is done in hypervisor, write the hook EIP in a magic MSR...
	* Make it working with AMD, 2 lines to change,
	* Could be integrated in (this nice tool) <a href="https://github.com/zer0mem/MiniHyperVisorProject">MiniHyperVisorProject</a>, to make it working on a live Windows (bluePill+Intercept R/W MSR+frookSINATRA = Rootkit ;p)
	* Real syscall analysis...but this is a Poc...
</pre>
