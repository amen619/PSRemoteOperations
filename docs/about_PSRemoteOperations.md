﻿# PSRemoteOperation

## about_PSRemoteOperations

## SHORT DESCRIPTION

This topic file explains the concepts and commands around a process called
PSRemoteOperations. The premise is that a computer is monitoring a given folder
looking for a file with a name that matches the computer name. When found, the
file can be parsed and the designated scriptblock or script file executed. Upon
completion, the original file is deleted and an archived copy is created which
contains metadata about the operation and its result.

## LONG DESCRIPTION

This module is predicated on the idea that you might have a computer or server
that utilizes some type of cloud service, such as DropBox or OneDrive. Or the
remote computer has access to a common folder such as a UNC path. The concept
is that you create a PSRemoteOperations file in "your" copy of the folder. The
folder is "replicated" to the remote computer.

The remote computer has some sort of "watcher" that is monitoring the folder.
When it detects a matching file, it is parsed and executed. The end result is
that you can invoke a command on a remote machine without relying on PowerShell
Remoting.

### Global Variables

It is strongly recommended that you define two global variables in your
PowerShell profile script. The commands in this module are designed to use the
variables as default.

`$PSRemoteOpPath` will be the path where the PSRemoteOperation files will be
created.

`$PSRemoteOpArchive` will be the path where the archive files will be created.
Typically, this will be a sub-folder of `$PSRemoteOpPath`.

```powershell
PS C:\> mkdir c:\users\jeff\dropbox\ops\archive
PS C:\> $PSRemoteOpPath = "c:\users\jeff\dropbox\ops\"
PS C:\> $PSRemoteOpArchive = "c:\users\jeff\dropbox\ops\archive"
```

If you use the global variables, the `Computername` parameter will autocomplete
for `Get-PSRemoteOperationResult` and `New-PSRemoteOperation` based on files in
the archive folder.

## EXAMPLES

With these default variables, here's how you might use the commands in this
module. First, you need to create an operation file.

```powershell
PS C:\> New-PSRemoteOperation -Computername SRV1 -Scriptblock {restart-service spooler -force}
```

This will create a file using the naming convention computername_guid.psd1.
It is assumed that the destination path, $PSRemoteOpPath, is being replicated
in some way to the remote computer, SRV1.

You can also create multiple remote operations file to run the same script or
scriptblock.

```powershell
PS C:\> $computers = Get-Content computers.txt
PS C:\> New-PSRemoteOperation -Computername $computers -Scriptblock {
    if (-Not (Test-Path C:\Work)) {
        mkdir c:\work
    }
    Copy-Item C:\Data\foo.dat -destination C:\work
}
```

This will create multiple psd1 files with the same scriptblock but for each
computer in the $Computers variable.

The remote computer needs some means of monitoring the target folder and
invoking the file when detected. You can use whatever means you want. You may
want to setup a FileWatcher or WMI Event subscription. You may have 3rd party
products you can use. However you monitor the folder, once you've identified a
matching file, use Invoke-PSRemoteOperation to execute it.

```powershell
PS C:\> Invoke-PSRemoteOperation $file -archivepath c:\archive
```

Once the operation is complete, the original file is deleted and an archive
version is created in the archive path location. If you placed your archive
as a sub-directory, the results will "replicate" back to you.

On your computer, you can use Get-PSRemoteOperationResult to get the results
from one or more computers or operations.

```powershell
PS C:\> Get-PSRemoteOperationResult -Computername SRV1 -Newest 1

        Computername  : SRV1
        Date          : 10/02/2018 21:29:35 UTC
        Scriptblock   : restart-service spooler -force
        Filepath      :
        ArgumentList  :
        Completed     : True
        Error         :
```

In this example, the command is getting new last result for computer SRV1.

It is up to you to manually manage the archive folder, deleting files as you
need to.

The module includes a command to setup a default "watcher" using a PowerShell
scheduled job.

```powershell
PS C:\> Register-PSRemoteOperationWatcher
```

You will be prompted for your user credentials. This will create a scheduled
job called RemoteOpWatcher that will check every 5 minutes for matching files
in `$PSRemoteOpPath` and use `$PSRemoteOpArchive` for the archive path. Use the
scheduled job cmdlets to modify or un-register. Note that the job won't start
for 2 minutes upon initial setup. But thereafter it will run indefinitely and
survive reboots.

## SECURITY

It is assumed that you are taking the necessary precautions to protect and
secure the locations for the PSREmoteOperations files. Presumably, if you are
using a cloud-based service like DropBox your files already have one layer of
protection. But if you are using something internal, such as a file share, you
will need to carefully watch access control.

Another option is to create the PSRemoteOperations files as protected CMS
messages as you would with the `Protect-CMSMessage` cmdlet. `New-PSRemoteOperation`
has a To parameter which takes a CMSMessageRecipient as a value. All other
commands will seamlessly detect if you are using a CMS Message or not and
automatically handle decryption.

This process will require the same documentation encryption certificate on the
system where you are creating the files as well as the remote computer. In
order to properly decrypt the message, the computer will need the private keys.
For the sake of simplicity, install the certificate with private keys on every
computer you intend to run remote operations.

CMS Messages are not supported on non-Windows platforms. Any CMS-related
parameters in this module are dynamic and ignored on non-Windows platforms.

## PowerShell Core

The long-term goal is to ensure that this module will work cross-platform and
in PowerShell Core. Support for CMS messages is limited to Windows platforms
through the use of dynamic parameters. `Register-PSRemoteOperationWatcher`
requires a Windows platform but should work under PowerShell Core. For
non-Windows systems, you will have to come up with your own tooling for
monitoring and execution using `Invoke-PSRemoteOperation`.

## NOTE

This module is intended to run local commands on a remote machine. Passing
credentials or running commands to remote machines where authentication might
be required is still under development and testing.

**TEST AND VERIFY ALL COMMANDS IN THS MODULE IN A NON-PRODUCTION ENVIRONMENT
ESPECIALLY IF USING DOCUMENT ENCRYPTION.**

## TROUBLESHOOTING NOTE

Please report any issues, bugs, comments or feature requests in the module's
GitHub repository at:

[https://github.com/jdhitsolutions/PSRemoteOperations/issues](https://github.com/jdhitsolutions/PSRemoteOperations/issues)

## SEE ALSO

`Protect-CMSMessage`

## KEYWORDS

`RemoteOperation`
