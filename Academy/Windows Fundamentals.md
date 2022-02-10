# Windows Fundamentals
### The Windows OS
Microsoft first introduced Windows on November 20th, 1985. The first version was a graphical OS shell for MS-DOS. Later versions of Windows Desktop introduced the Windows File Manager, Program Manager, and Print Manager programs.

Windows 95 was the first full integration of Windows and DOS and offered built-in Internet support for the first time. This version also debuted the Internet Explorer web browser. Since the intitial version, there have been over a dozen versions of Windows released such as XP, Vista, 8, 10.

Windows Server first released in 1993 with the release of Windows NT 3.1 Advanced Server. Windows NT saw several updates over the years, adding the technologies such as Internet Informatin Services (IIS), various networking protocols, Administrative Wizards to facilitate admin tasks, and more. The release of Windows 2000, Active Directory was debuted, originally intended for sysadmins to help them set up file sharing, data encryption, VPNs, etc. 

### Windows Versions
List of major Windows OSs and associated version numbers:
- Windows NT 4: 4.0
- Windows 2000: 5.0
- Windows XP: 5.1
- Windows Server 2003, 2003 R2: 5.2
- Windows Vista, Server 2008: 6.0
- Windows 7, Server 2008 R2: 6.1
- Windows 8, Server 2012: 6.2
- Windows 8.1, Server 2012 R2: 6.3
- Windows 10, Server 2016, Server 2019: 10.0

You can use the `Get-WmiObject` cmdlet to find informatin about the operating system.

`Get-WmiObject -Class win32_OperatingSystem`

Some other useful classes are `win32_Process` to get a process listing, `win32_service` to get a listing of services, `win32_Bios` to get BIOS information. We can use the `ComputerName` parameter to get information about remote computers. 

### OS Structure
The root directory is `<drive_letter>:\(commonly C drive)`. The root directory (also known as boot partition) is where the OS is installed. Other physical and virtual drives are assigned other letters, for example, Data (E:). The directory structure of the boot partition is as follows:

- Perflogs: Can hold Windows performance logs but is empty by default
- Program Files:  On 32-bit systems, all 16-bit and 32-bit programs are installed here. On 64-bit systems, only 64-bit programs are installed here.
- Program Files (x86): 32-bit and 16-bit programs are installed here on 64-bit systems.
- ProgramData: Hidden folder containing data that is essential for certain installed programs to run.
- Users: Contains user profiles for each user that logs onto the system and contains the two folders Public and Default.
- Default: This is the default user profile template for all created users. Whenever a new user is added to the system, their profile is based on the Default profile.
- Public: This folder is intended for computer users to share files and is accessible to all users by default. This folder is shared over the network by default but requires a valid network account to access.
- AppData: Hidden user subfolder. Each of these folders contains three subfolders. The Roaming folder contains machine-independent data that should follow the user's profile, such as custom dictionaries. The Local folder is specific to the computer itself and is never synchronized across the network. LocalLow is similar to the Local folder, but it has a lower data integrity level.
- Windows: Majority of the files required for the Windows OS are contained here.
- System, System32, SysWOW64: Contains all DLLs required for the core features of Windows and the Windows API. The OS searches these folders any time a program asks to load a DLL without specifying an absolute path.
- WinSxS: The Windows Component Store contains a copy of all Windows components, updates and service packs.

### Exploring Directories Using Command Line
We can explore the file system using the `dir` command

The `tree` utility is useful for graphically displaying the directory structure of a path or disk. Can provide us with a large amount of information. The following command can be used to walk through all the files in the C drive, one screen at a time.

`tree c:\ /f | more`

### File System
There are 5 types of Windows file systems: FAT 12, FAT 16, FAT32, NTFS, and exFAT. FAT12 and FAT16 are no longer used on modern Windows OSs.

FAT32 (File Allocation Table) is widely used across many types of storage devices such as USB memory sticks and SD cards but can also be used to format hard drives. The "32" in the name refers to the fact that FAT32 uses 32 bits of data for identifying data clusters on a storage device.

`Pros of FAT32`:
- Device compatibility: Can be used on computers, digital cameras, gaming consoles, smart phones and more
- OS cross-compatibility: works on all Windows OSs starting from Windows 95 and is also supported by MacOS and Linux

`Cons of FAT32`:
- Can only be used with files that are less than 4GB
- No built-in data protection or file compression features
- Must use 3rd party tools for file encryption

NTFS (New Technology File System) is the default Windows file system since Windows 3.1. NTFS also has better support for metadata and better performance due to improved data structuring.

`Pros of NTFS`:
- NTFS is reliable and can restore the consistency of the file system in the event of a system failure or power loss.
- Provides security by allowing us to set granular permissions on both files and folders.
- Supports very large-sized partitions
- Has journaling built-in, meaning that file modifications (addition, mofification, deletion) are logged

`Cons of NTFS`:
- Most mobile devices do not support NTFS natively
- Older media devices such as TVs and digital cameras do not offer support for NTFS storage devices

### Permissions
The NTFS file system has many basic and advanced permissions. Some of the key permissions are:
- **Full Control**: Allows reading, writing, changing, deleting of files/folders
- **Modify**: Allow reading, writing, and deleting of files/folders
- **List Folder Contents**: Allows for viewing and listing folders and subfolders as well as executing files. Folders only inherit this permission.
- **Read and Execute**: Allows for viewing and listing files and subfolders as well as executing files. Files and folders inherit this permission
- **Write**: Allows for adding files to folders and subfolders and writing to a file
- **Read**: Allows for viewing and listing of folders and subfolders and viewing a file's contents
- **Traverse Folder**: This allows or denies the ability to move through folders to reach other files or folders.

Files and folders inherit the NTFS permissions of their parent folder for ease of administration, so admins do not need to explicity set permissions for each file and folder, as this would be extremely time-consuming.

### Integrity Control Access List (icacls)
NTFS permissions on files and folders in Windows can be managed using the File Explorer GUI under the security tab. Apart from the GUI, we can also achieve a fine level of granularity over NTFS file permissions in Windows from the command line using the icacls utility.

We can list out the NTFS permissions on a specific directory by running either `icacls` from within working directory or `icacls C:\Windows` against a directory not currently in.

The resource access level is listed after each user in the output. The possible inheritance settings are:
- `(CI)`: Container Inherit
- `(OI)`: Object Inherit
- `(IO)`: Inherity Only
- `(NP)`: Do no propagate inherit
- `(I)`: permission inherited from parent container

In the above example, the `NT AUTHORITY\SYSTEM` account has object inherit, container inherit, inherity only, and full access permissions. This means that this account has full control over all file system objects in this directory and subdirectories

Basic access permissions are as follows:
- `F`: full access
- `D`: delete access
- `N`: No access
- `M`: Modifiy access
- `RX`: Read and execute access
- `R`: Read only access
- `W`: Write-only access

You can add and remove permissions via the command line using `icacls`. Here we are executing `icacls` in the context of a local administrator account showing the `C:\users` directory where the `joe` user does not have any write permissions

Using the command `icacls C:\Users /grant joe:f` will grant the user `joe` full control over the directory, but given the `(oi)` and `(ci)` were not included in the command, the joe user will only have rights over the `C:\Users` folder but not over the user subdirectories and files contained within them.

These permissions can be revoked using the command `icacls C:\users /remove joe`

`icacls` is a very powerful tool and can be used in a domain setting to give certain users or groups specific permissions over a file or folder, explicity deny, access, enable or disable inheritance permissions, and change directory/file ownership.

### NTFS vs. Share Permissions
Microsoft owns over 70% of the global market share on desktop OSs with Windows. Many variants of malware written for Windows can spread over the network via network shares with lenient permissions applied. It is also worth noting that to this day, the infamous `EternalBlue` vulnerability still haunts unpatched Windows systems running `SMBv1` and often paves the way for ransomware to shut down organizations.

The `Server Message Block (SMB)` is used in Windows to connect shared resources like files and printers. It is used in large, medium and small enterprise environments.

![[Pasted image 20220209111017.png]]

NTFS permissions and share permissions are often understood to be the same, but they are not however, they can apply to the same shared resource.

**Share Permissions**
- `Full Control`: Users are permissed to perform all actions given by Chang and Read permissions as well as change permisions for NTFS files and subfolders.
- `Change`: Users are permitted to read, edit delete and add files and subfolders.
- `Read`: Users are allowed to view file & subfolder contents.

**NTFS Basic permissions**
- `Full Control`: Users are permitted to add, edit, move, delete files & folders as well as change NTFS permissions that apply to all allowed folders.
- `Modify`: Users are permitted or denied to view and modify files and folders. This includes adding or deleting files
- `Read & Execute`: users are permitted or denied to read the contents of files and execute programs
- `List folder contents`: view a listing of files and subfolders
- `Read`: read contents of the file
- `Write`: write changes to a file and add new files to a folder
- `Special Permissions`: A variety of advanced permissions options

**NTFS special permissions**
Full control, traverse folder/execute file, list folder/read data, read attributes, read extended attributes, crete files/write data, create folders/append data, write attributes, write extended attributes, delete subfolders and files, delete, read permissions, change permissions, take ownership.

### Creating a Network Share
To create a share on the `Windows 10 target box`.

### Windows Services & Processes
Services are a major component of the Windows OS. They allow for the creation and management of long-running processes. Windows services can be started automatically at system boot without user intervention. These services can continue to run in the background even after the user logs out of their account on the system.

Applications can also be created to install as a service, such as a network monitoring application installed on a server. Services on Windows are responsible for many functions within the Windows OS, such as networking functions, performing system diagnostics, managing user credentials, controlling Windows updates, and more.

Windows services are managed via the Service Control Manager (SCM) system, accessible via the `services.msc` MMC add-in.

The information includes the service Name, Description, Status, Startup Type, and the user that the service runs under.

It is also possible to query and manage services via the command line using `sc.exe` using **PowerShell** cmdlets such as `Get-Service`

Ex) `Get-Service | ? {$_.Status -eq "Running"} | select -First 2 | fl`

Service statuses can appear as Running, Stopped or Paused, and they can be set to start manually, automatically, or on a delay at system boot. Services can also be shown in the state of Starting or Stotpped if some action has triggered them to either start or stop. Windows has three categories of services: Local Services, Network Services, and System Services. Services can usually only be created, modified, and deleted by users with admnistrative privileges. Misconfigurations around service permissions are a common privilege escalation vector on Windows systems.

**Critical system services**
- smss.exe: Session Manager SubSystem. Reponsible for handling sessions on the system.
- csrss.exe: Client Server Runtime Process. The user-mode portion of the Windows subsystem.
- wininit.exe: Starts the Wininit file .ini file that lists all of the changes to be made to Windows when the computer is restarted after installing a program.
- logonui.exe: used for facilitating user login into a PC
- lsass.exe: Local Security Autentication Server verifies the validity of user logons to a PC or server. It generates the process responsible for authenticating users for the Winlogon service.
- services.exe: manges the operation of starting and stopping services
- winlogon.exe: Responsible for handling the secure attention sequence, loading a user profile on logon, and locking the computer when a screensaver is running
- System: A background system process that runs the Windows Kernel
- svchost.exe w/ RPCSS: Manages system services that run from DLLs such as "Automatic Updates", "Windows Firewall", and "Plug and Play". Uses the Remote Procedure Call (RPC) Service (RPCSS)
- svchost.exe w/ Dcom/PnP: Manages system services that run from DLLs, uses the Distributed Component Object Model (DCOM) and Plug and Play (PnP)


### Services Permissions
Services allow for the management of long-running processes and are a critical part of Windows OSs. Sysadmins often overlook them as potential threat vectors that can be used to load malicious DLLs, execute applications without access to an admin account, escalate privileges and even maintain persistence.

**Examining Services using services.msc**
We can use `services.msc` to view and manage just about every detail regarding all services. Let's take a closer look at the service associated with `Windows update (wuauserv)`

Most services run with LocalSyste privileges by default which is the highest level of access allowed on an individual Windows OS. Not all applications need Local System account-level permissions, so it is beneficial to perform research on a case-by-case basis when considering installing new applications in a Windows environment. It is a good practice to identify applications that can run with the least privileges possible to align with the principle of least privilege.

Notable built-in service accounts in Windows:
- LocalService
- NetworkService
- LocalSystem

### Examining services using sc
sc.exe can also be used to configure and manage services.

`sc qc wuauserv`

The `sc qc` command is used to query the service. This is where knowing the names of the services can come in handy. If we wanted to query a service on a device over the network, we could specify the hostname of IP address immeditately after `sc`

The `sc config <service> binPath=<path>` can be used to change the associated binary path to the service and could be replaced with something malicious.

`sc sdshow <service>`

Every named object in Windows is a securable object, and even some unamed objects are securable. If it's securable in a Windows OS, it will have a security descriptor. Security descriptors identify the object's owner and a primary group containing `Discretionary Access Control List (DACL)` and a `System Access Control List (SACL)`.

Generally, a DACL is used for controlling access to an object, and a SACL is used to account for and log access attempts.

The amalgamation of characters crunched together and deliminated by opened and closed parenthesis is in a format known as the `Security Descriptor Definition Language (SDDL)`.

`D: (A;;CCLCSWRPLORC;;;AU)`

1. D: - the proceeding characters are DACL parameters
2. AU - defines the security principal Authenticated Users
3. A;; - access is allowed
4. CC - SERVICE_QUERY_CONFIG is the full name, and it is a query to the service control manager (SCM)
5. LC - SERVICE_QUERY_STATUS, it is a query to the SCM for the current status
6. SW - SERVICE_ENUMERATE_DEPENDENTS, will enumerate a list of dependent services
7. RP - SERVICE_START and will start the service
8. LO - SERVICE_INTERRROGATE and will query the service for its current status
9. RC - READ_CONTROL will query the security descriptor of the service.


### Windows Sessions
**Interactive**
An interactive, or local logon session, is initiated by a user authenticating to a local or domain system by entering their credentials. An interactive logon can be initiated by loggin directly into the system, by requesting a secondary logon session using the `runas` command via the command line, or througha Remote Desktop connection.

**Non-interactive**
Differ from standard user accounts as they do not require login credentials. There are 3 types of non-interactive accounts: Local System, Local Service, and the Network Service accounts. Are generally used by the Windows OS to automatically start services and applications without requiring user interaction. These accounts have no password associated with them and are usually used to start services when the system boots or to runs scheduled tasks.

- `NT AUTHORITY\SYSTEM`
- `NT AUTHORITY\LocalService`
- `NT AUTHORITY\NetworkService`






