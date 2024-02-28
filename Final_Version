<#
.Synopsis
  cdw-getUSERGroup.ps1 - script to display all users and groups installed on system   
.DESCRIPTION
  cdw-getUSERGroup.ps1 - script to display all users and groups installed on system
.EXAMPLE
  executed on a local system taking defaults ([return]):
    > .\cdw-getUSERgroup.ps1 [return]  - default execution that displays output to the screen
.EXAMPLE
    > . .\cdw-getUSERgroup.ps1 [return]  - default execution that displays output to the screen
.EXAMPLE
    > getUSERgroup -report y [return]  - parameter that saves the output to the machine executing the script
.EXAMPLE
  executed on a local system:
    > .\cdw-getUSERgroup.ps1 [return]
    [output] - review output written to screen to view all user and groups that are installed within the system/group(s)    
  executed on a remote system taking defaults ([return]):
.EXAMPLE
    >  icm -FilePath .\cdw-getUSERgroup.ps1 -cn TestSERVER10.abc.net (fqdn) [return]
.EXAMPLE
  remotely executed with an 'Administrative PowerShell' session with the script located in current directory and saving the output to c:\temp:
    > icm -cn ("TestSERVER10.abc.net","TestSERVER11.abc.net") -FilePath .\cdw-getUSERgroup.ps1 | tee-object -FilePath .\zzDELETE123_TRN01A_TRN01B.txt
.INPUTS
  boolean computername, report for local execution
.OUTPUTS
  Script displays all local user and group information either within the session the script was executed or if the report parameter was set the output
  will be displayed on the screen and within the system the script was executed on 'C:\temp' (or if the temp folder does not exists the report is 
  written to the current folder location the script was executed from (for example, C:\Users\NTaccount\).
.NOTES
Source websites used to making this a functional script:
  icm command(s)         https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-7.2&viewFallbackFrom=powershell-6
  $var = @(foreach...    Appending [pscustomobject] variable:  https://stackoverflow.com/questions/47096939/powershell-looping-and-storing-content-to-variable
.COMPONENT
   The component this cmdlet belongs to Jeff Giese
.ROLE
  The role this cmdlet belongs to Corporate Data Warehouse System Administrator Team (CDW SA Team)
.FUNCTIONALITY
  Script to gather local or remote User and Group account listings within the 'Computer Management Console \ Local User and Groups' (compmgmt.msc)
#>
function cdw-getUSERgroup(){
    [CmdletBinding()]
    [Alias("getUSERgroup")]
    Param(
        [parameter(mandatory=$false,
        valuefrompipeline=$true)]
        [bool]$computername,
        [bool]$report
    )

    switch($computername){
        $true{
            $fqdn_example = (($env:COMPUTERNAME) + "." + $env:USERDNSDOMAIN)
            Write-Output "(ensure you are within the directory where the 'cdw-getUSERgroup.ps1' script resides)"

            do{
                $fqdn = read-host " fqdn of remote system to view local user and group information ($($fqdn_example))"
                
                $fqdn_verify = read-host " $($fqdn): `t - correct? (y/n)"
                $fqdn
            }until($fqdn_verify -eq "y")

#NOTE: when loading the script into memory via dot-sourcing ('. .\...'), if you are not executing the script within the directory/folder location
#where the script resides and running on a remote system via the param '-computername 1' the script will error out with a '... does not exist.'
#message. To fix, change directory to the location of where you copied the script over to and then re-execute and it should run to completion.

            icm -cn $fqdn -filepath .\cdw-getUSERgroup.ps1
            write "end of remote report on:`t$($fqdn)"
            return
        }
        default{
            Continue
        }
    }

    $cn = $env:COMPUTERNAME
    $ea_b4 = $ErrorActionPreference
    $dir_b4 = $pwd.path
    $ErrorActionPreference = "silentlycontinue"
    [datetime]$date = get-date
    $HourMinDATE = $date.ToString("HH" + "mm" + "__yyyy_MM_dd")
    $localgroup = Get-LocalGroup | select -ExpandProperty name | sort
    $localgroupREPORT = ""
    $report_DIR = ""
    $report_DIR = "C:\temp"
    $report_HT = [ordered]@{}
    $scriptEXE_localHOST = ($env:COMPUTERNAME)
    $scriptEXE_NTacct = ($env:USERDOMAIN + "\" + $env:USERNAME)

#populating HT for reporting purposes:
    $report_HT += @{
        date_timestamp = $date
        script__name = "cdw-getUSERgroup"
        script_description = "user/group account listing from 'Computer Mgmt Console\Local User and Groups' (compmgmt.msc)"
        script_executed_NTacct = $scriptEXE_NTacct
        script_executed_SERVER = $scriptEXE_localHOST
    }

    $localgroupREPORT = @(foreach($l in $localgroup){
            [PSCustomObject]@{
                GroupNAME = (@(get-localgroup $l | select -ExpandProperty name) -join ',')
                GroupDESCRIPTION = (@(Get-LocalGroup $l | select -ExpandProperty description) -join ',')
                GroupSID  = (@(Get-LocalGroup $l | select -ExpandProperty sid) -join ',')
                GroupOBJECTCLASS = (@(Get-LocalGroup $l | select -ExpandProperty objectclass) -join ',')
                GroupMemberNAME = (@(Get-LocalGroupMember $l | select -ExpandProperty name | sort name) -join ', ')
                GroupMemberSID = (@(Get-LocalGroupMember $l | select -ExpandProperty sid)  -join ', ')
                GroupMemberPRINCIPALSOURCE = (@(Get-LocalGroupMember $l | select -ExpandProperty principalsource)  -join ', ')
                GroupMemberOBJECTCLASS = (@(Get-LocalGroupMember $l | select -ExpandProperty ObjectClass)  -join ', ')
            }
        }
    )

    clear-host

    if($report -eq $false){
        write-output "$($date::now())"
        Write-Host -ForegroundColor Green " $($cn) : Local User and Group Report output to screen"
        $report_HT.GetEnumerator() | select name,value | ft -AutoSize -HideTableHeaders
        $report = $false
        $report_DIR = $pwd.Path
        $localgroupREPORT | select @{n="GROUP NAME ON:  '$($scriptEXE_localHOST)'";e={$_.groupname}},@{n="USERS ON:  '$($scriptEXE_localHOST)'";e={$_.groupmembername}} | ft -Wrap -AutoSize
        sl $dir_b4
        $ErrorActionPreference = $ea_b4
    }
    else{
        $report_check = test-path $report_DIR
        if($report_check -ne $true){
            $report_DIR = $pwd.path  
        }
        else{
            $report_DIR = "c:\temp"
        }

        Write-host -ForegroundColor Green " report parameter set with report saved to:`t'$report_DIR':"
        write-output "$($report_all)"
        $cn = $env:COMPUTERNAME

        sl $report_DIR
        $localgroupREPORT | export-csv -NoTypeInformation ("USERgroupReport__" + "$($cn)" +  "____" + "$([datetime]::Now.ToString("HH" + "mm" + "__yyyy_MM_dd"))" + ".csv") -Force -Verbose
        ii $report_DIR | sort lastwritetime -Descending
        
        sl $dir_b4
        $ErrorActionPreference = $ea_b4
    }
} #End of function

write "Executing script:`t'cdw-getUSERgroup'`n"
Start-Sleep -Seconds 1
getUSERgroup
