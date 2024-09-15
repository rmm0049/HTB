#### Intro to metasploit
- `Metasploit` is a Ruby-based, modular, penetration testing platform that enables you to write, test and execute the exploit code.
The `modules` mentioned are actual exploit PoC that have already been developed and tested in the wild and integrated within the framework to provide pentesters with ease of access to different attack vectors for different platforms and services.

##### Metasploit Pro

`Metasploit` as a product is split into two versions.  It has some additional features:
- Task Chains
- Social Engineering
- Vulnerability Validations
- GUI
- Quick Start Wizards
- Nexpose Integreation

##### Understanding the Architecture

The default location of the MSF is in `/usr/share/metasploit-framework`

The *Data* and *Lib* are the functioning parts of the msfconsole interface, while the Documentation folder contains all the technical details about the project

### Intro to MSFconsole

`msfconsole` in the terminal will launch the framework.

To not display the banner
`msfconsole -q`

To update
`msfupdate`
or
`sudo apt update`

##### MSF Engagement Structure

The MSF engagement structure can be divided into five main categories
1. Enumeration
2. Preparation
3. Exploitation
4. Privilege Escalation
5. Post-Exploitation

The division makes it easier to find and select the appropriate MSF features in a more strucutred way and work accordingly

### Modules

The `exploit` cateogory consists of so-called proof-of-concept (`PoCs`) that can be used to exploit exisiting vulnerabilities in a largely automated manner. Many exploits require customization and so just because it doesn't work right away, doesn't mean a target is not vulnerable.

**Syntax**

`<No.> <type>/<os>/<service>/<name>`

**Example**

`794 exploit/windows/ftp/scriptftp_list`

The `type` tag is the first level of segregation between the Metasploit `modules`. Looking at this field, you can tell what the piece of code for this module will accomplish.

`Auxiliary` -> scanning, fuzzing, sniffing
`Encoders` -> ensures payloads are intact to their destination
`Exploits` -> Defined as modules that exploit a vulnerability that will allow for payload delivery
`NOPs` -> Keep the payload size consistent across exploit attempts
`Payloads` -> Code runs remotely and calls back to attacker machine
`Plugins` -> Additional scripts can be integrated within an assessment with `msfconsole` and coexist
`Post` -> wide arrary of modules to gather info, pivot deeper, etc.

**Searching**

`search eternalromance`

**Specific Search**

`search type:exploit platform:windows cve:2021 rank:excellent microsoft`

### Targets

The `show targets` command issued within an exploit module view will display all available vulnerable targets for that specific exploit, while issuing the same command in the root menu, outside of any selected exploit module, will let us know that we need to select an exploit module first