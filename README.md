# getGROUPinfo
  cdw-getUSERGroupReport.ps1 - script to display all users and groups installed on system
PowerShell script that starts on line 5 with the comment "<#" thru line 138 (I'm new to GitHub so this may all need redone eventually...)

<#
2024-1-31  cdw-getUSERgroup.ps1 script
.Synopsis
  cdw-getUSERGroup.ps1 - script to display all users and groups installed on system   
.DESCRIPTION
  cdw-getUSERGroup.ps1 - script to display all users and groups installed on system
.EXAMPLE
  executed on a local system taking defaults ([return]):
    > .\cdw-getUSERgroup.ps1
              [return]  - default execution that displays output to the screen
    -report y [return]  - parameter that saves the output to the machine executing the script, 
                          currently only available for local executions of the script
.EXAMPLE
  executed on a local system:
    > .\cdw-getUSERgroup.ps1
    [output] - review output written to screen to view all user and groups that are installed within the system/group(s)    
  executed on a remote system taking defaults ([return]):
    .> icm -FilePath .\cdw-getUSERgroup.ps1 -cn TestSERVER10.abc.net (fqdn) [return]

.INPUTS
  drive, path, and report paramaters
.OUTPUTS
  Script displays all local user and group information either within the session the script was executed or if the report parameter was set the output
  will be displayed on the screen and within the system the script was executed on 'C:\temp' (or if the temp folder does not exists the report is 
  written to the current folder location the script was executed from (for example, C:\Users\NTaccount\).
.NOTES
Source websites used to making this a functional script:
  icm command(s)         https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-7.2&viewFallbackFrom=powershell-6
.COMPONENT
   The component this cmdlet belongs to Jeff Giese
.ROLE
  The role this cmdlet belongs to Corporate Data Warehouse System Administrator Team (CDW SA Team)
.FUNCTIONALITY
  Script to gather local or remote User and Group account listings within the 'Computer Management Console \ Local User and Groups' (compmgmt.msc)
#>

#$localgroup = get-localgroup | select -ExpandProperty name
function cdw-getUSERgroup(){
    [CmdletBinding()]
    [Alias("getUSERgroup")]
    Param(
        [parameter(mandatory=$false,
        valuefrompipeline=$true)]
        [string]$report
    )

    $cn = $env:COMPUTERNAME
    $ea_b4 = $ErrorActionPreference
    $ErrorActionPreference = "silentlycontinue"
    [datetime]$date = get-date
    $HourMinDATE = $date.ToString("HH" + "mm" + "__yyyy_MM_dd")
    $localgroup = ""
    $localgroup = Get-LocalGroup | select -ExpandProperty name | sort
#$localgroup = get-localgroup | select @{n="name";e={$_.name -as [string]}} | sort name
    $localgroup2_screen = [ordered]@{}
    $report2_screen = [ordered]@{}
    $report_DIR = ""
    $report_DIR = "C:\temp"
    $report_HT = [ordered]@{}
    $scriptEXE_localHOST = ($env:COMPUTERNAME + "." + $env:USERDNSDOMAIN) #system running/executing the script
    $scriptEXE_NTacct = ($env:USERDOMAIN + "\" + $env:USERNAME)

#Remote execution runs - but i need to fix the writing to the screen for output...
#make notes for   icm command for remote runs
#> icm -FilePath .\cdw-getGROUPinfo__v3.ps1 -cn vhacdwdwhtrn01b.vha.med.va.gov
#Executing script:       'cdw-getUSERgroup'
#
#
#01/29/2024 16:36:33
#
#
#WARNING:  Report PARAM not set!!!!

#populating HT for reporting purposes:
    $report_HT += @{
        date_timestamp = $date
        script__name = "cdw-getUSERgroup"
        script_description = "user/group account listing from 'Computer Mgmt Console\Local User and Groups' (compmgmt.msc)"
        script_executed_NTacct = $scriptEXE_NTacct
        script_executed_SERVER = $scriptEXE_localHOST
    }

    new-item -ItemType file -Path $report_DIR\USERgroupReport.csv -Force
    foreach($l in $localgroup){
        [PSCustomObject]@{
            GroupNAME = (@(get-localgroup $l | select -ExpandProperty name) -join ',')
            GroupDESCRIPTION = (@(Get-LocalGroup $l | select -ExpandProperty description) -join ',')
            GroupSID  = (@(Get-LocalGroup $l | select -ExpandProperty sid) -join ',')
            GroupOBJECTCLASS = (@(Get-LocalGroup $l | select -ExpandProperty objectclass) -join ',')
            GroupMemberNAME = (@(Get-LocalGroupMember $l | select -ExpandProperty name | sort name) -join ', ')
            GroupMemberSID = (@(Get-LocalGroupMember $l | select -ExpandProperty sid)  -join ', ')
            GroupMemberPRINCIPALSOURCE = (@(Get-LocalGroupMember $l | select -ExpandProperty principalsource)  -join ', ')
            GroupMemberOBJECTCLASS = (@(Get-LocalGroupMember $l | select -ExpandProperty ObjectClass)  -join ', ')
        } | export-csv -NoTypeInformation $report_DIR\USERgroupReport.csv -Append
    }
    $report_all = ipcsv $report_DIR\USERgroupReport.csv

    clear-host

#if report is NULL, then output is to the screen
    if(([string]::IsNullOrWhiteSpace($report))){
        write-output "$($date::now())"
        Write-Warning " $($cn) : Local User and Group Report output to screen"
        $report_HT.GetEnumerator() | select name,value | ft -AutoSize -HideTableHeaders
        $report = $false
        $report_DIR = $pwd.Path
        $report_all | select @{n="GROUP NAME ON:  '$($cn)'";e={$_.groupname}},@{n="USERS ON:  '$($cn)'";e={$_.groupmembername}} | ft -Wrap -AutoSize
        sleep -Milliseconds 200

        remove-item $report_DIR\USERgroupReport.csv
    }
#Report param set
    else{
        Write-host -ForegroundColor Green " Report PARAM SET!"
        $report = $true
        $report_check = test-path $report_DIR
        if($report_check -ne $true){
            $report_DIR = $pwd.path
        }
        else{
            $report_DIR = "c:\temp"
        }
        write-output "$($report_all)"
        $cn = $env:COMPUTERNAME
        rename-item $report_DIR\USERgroupReport.csv ("USERgroupReport__" + "$($cn)" +  "____" + "$([datetime]::Now.ToString("HH" + "mm" + "__yyyy_MM_dd"))" + ".csv")
        ii $report_DIR | sort lastwritetime -Descending
    }


} #End of function

write "Executing script:`t'cdw-getUSERgroup'`n"
Start-Sleep -Seconds 2
getUSERgroup
