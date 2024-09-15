The built-in command shell `CMD.exe` and PowerShell are two implementations included in all Windows hosts.

##### Command Prompt vs. PowerSehll

To run PowerShell commands from cmd.exe you would have to preface the command with `powershell`

![[Pasted image 20230301202512.png]]

## Command Prompt Basics

Known as `cmd.exe` or CMD, is the default command line interpreter for the WIndows OS. Originally based on the `COMMAND.COM` interpreter in DOS, the Command prompt is ubiquitous across nearly all Windows OSs.

**`Quick Story:`** Several times during a pentest, I have run into hosts that had PowerShell locked down pretty well or made completely inaccessible through application control such as AppLocker. Using the Command Prompt, I could still leverage the host to acquire further access and elevate my privileges to continue the assessment. Modern operating systems have plenty of legacy software still embedded within the hosts. As admins and assessors alike, we must be aware of this and understand how to use them to our advantage.

#### Accessing CMD

**Local Access vs. Remote Access**

Local Access

- Windows key + `r` to run prompt, then `cmd` OR
- Accessing the executable in `C:\Windows\System32\cmd.exe`

Remote Access

- Telnet
- ssh
- PsExec
- WinRM
- RDP
- ...

Windows installation disc gives us the option to boot into `Repari Mode` You can access a command prompt here

While useful, this also poses a potential risk. For example, on this Windows 7 machine, we can use the recovery Command Prompt to tamper with the filesystem. Specifically, replacing the `Sticky Keys` (`sethc.exe`) binary with another copy of `cmd.exe`

Once the machine is rebooted, we can press `Shift` five times on the Windows login screen to invoke `Sticky Keys`. Since the executable has been overwritten, what we get instead is another Command Prompt - this time with `NT AUTHORITY\SYSTEM` permissions. We have bypassed any authentication and now have access to the machine as the super user.


## Getting Help

cmd has a built-in `help` function that can help with detailed info about the available commands on the system

Using `help` by itself will list out system commands (`built-ins`) and provides a basic description of its functionality

Then you can use `help <command name>`

Some commands you need to use the `/?` modifier to provide the help like `ipconfig /?`

##### Basic tips and tricks

`cls` to clear the screen/terminal window

`doskey /history` keeps a history of commands and allows them to be referenced again

`Ctrl+c` to interrupt the process happening

![[Pasted image 20230301204938.png]]

#### Interesting Directories

![[Pasted image 20230301205316.png]]

#### Directories

`mkdir` or `md`
`rmdir` or `rd`

`move <source> <dest>`

`xcopy <source> <dest> options` -> can remove the read only bit when copying

It has been deprecated for `robocopy`

