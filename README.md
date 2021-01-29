# CIMplant

C# port of WMImplant which uses either CIM or WMI to query remote systems. It can use provided credentials or the current user's session.

Note: Some commands will use PowerShell in combination with WMI, denoted with **.

## Introduction

CIMplant is a C# rewrite and expansion on [@christruncer](https://twitter.com/christruncer)'s [WMImplant](https://github.com/FortyNorthSecurity/WMImplant). It allows you to gather data about a remote system, execute commands, exfil data, and more. The tool allows connections using Windows Management Instrumentation, [WMI](https://docs.microsoft.com/en-us/windows/win32/wmisdk/about-wmi), or Common Interface Model, [CIM](https://www.dmtf.org/standards/cim) ; well more accurately Windows Management Infrastructure, [MI](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/wmi_v2/windows-management-infrastructure). CIMplant requires local administrator permissions on the target system.

## Usage

```
CIMplant.exe --help
CIMplant.exe --show-commands
CIMplant.exe --show-examples
CIMplant.exe -s [remote IP address] -c cat -f c:\users\user\desktop\file.txt
CIMplant.exe -s [remote IP address] -u [username] -d [domain] -p [password] -c cat -f c:\users\test\desktop\file.txt
CIMplant.exe -s [remote IP address] -u [username] -d [domain] -p [password] -c command_exec --execute "dir c:\\"
```
### Some Helpful Commands

![image](https://github.com/FortyNorthSecurity/CIMplant/raw/master/Extras/CIMplant-Help.gif)

### Some Example Usage Commands

![image](https://github.com/FortyNorthSecurity/CIMplant/raw/master/Extras/CIMplant-Usage.gif)

## Important Files

1. Program.cs
> This is the brains of the operation, the driver for the program.

2. Connector.cs
> This is where the initial CIM/WMI connections are made and passed to the rest of the application

2. ExecuteWMI.cs
> All function code for the WMI commands

3. ExecuteCIM.cs
> All function code for the CIM (MI) commands

## Detection

Of course, the first thing we'll want to be aware of is the initial WMI or CIM connection. In general, WMI uses DCOM as a communication protocol whereas CIM uses WSMan (or, WinRM). This can be modified for CIM, and is in CIMplant, but let's just go over the default values for now. For DCOM, the first thing we can do is look for initial TCP connections over **port 135**. The connecting and receiving systems will then decide on a new, very high port to use so that will vary drastically. For WSMan, the initial TCP connection is over **port 5985**.

Next, you'll want to look at the Microsoft-Windows-WMI-Activity/Trace event log in the Event Viewer. Search for **Event ID 11** and filter on the IsLocal property if possible. You can also look for **Event ID 1295** within the Microsoft-Windows-WinRM/Analytic log.

Finally, you'll want to look for any modifications to the **DebugFilePath** property with the **Win32_OSRecoveryConfiguration** class. More detailed information about detection can be found at our blog here: [CIMplant Part 1: Detection of a C# Implementation of WMImplant](https://fortynorthsecurity.com/blog/) (**UPDATE WHEN POSTED**)