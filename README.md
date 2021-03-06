# Migrate_PrintQs.vbs


Author       : Cary GARVIN  
Contact      : cary(at)garvin.tech  
LinkedIn     : [https://www.linkedin.com/in/cary-garvin](https://www.linkedin.com/in/cary-garvin)  
GitHub       : [https://github.com/carygarvin/](https://github.com/carygarvin/)  


Script Name  : [Migrate_PrintQs.vbs](https://github.com/carygarvin/Migrate_PrintQs.vbs)  
Version      : 1.0  
Release date : 07/02/2015 (CET)  

History      : The present Script has been used by large organizations to successfully migrate tens of thousands of network printers from old to new Print Servers. A lot of safeguards have been built into this Script.  

Purpose      : The present Script is to be used in the scope of a Print Server migration whereby Network Print Queues are migrated from one old Print Server (to be decommissioned) to a new Print server. The script will take care of remapping all of user Print Queues based on information contained in a mappings or correspondence file with each line in the format
                      
                          \\OldPrintServer\OldPrintQueueName,\\NewPrintServer\NewPrintQueueName

In addition, the Script has the ability to remove (Remove) some printers altogether or Add (Affix) one or more printers deemed essential.  

# Script information
Script to migrate user printers from one Print Server to another based on a correspondence/mapping file holding Old Print Queue to New Print Queue mappings.  
The present Script is best invoked during the Login Script through Active Directory Group membership or interactively by specifying any specific mapping file to use as a parameter. The present Script uses for maximum reliability several methods in order to identify user printers.  
The present Script has many features as follows:  
* Migrate user PrintQueues at logon or interactively based on information contained in the specified mappings file  
* Add one or more essential printers to which all users need to have access.  
* Unequivocally remove obsolete printers.  

As stated, the present Script can be invoked either from a Logon Script in which the current user may have its group membership tested and, if validated, call the Script. By default, the mappings file to be used will match the user's OU. This allows department/BU specific mappings file in case of large Organizations.  
Here's an example of how it can be called from within a "parent" VBScript Logon Script provided the Group's Distinguished Name is stored in **strGroupDN** and a binding has been made to the user object through **objUser**:  
  
                          Set objGroup = GetObject("LDAP://" & strGroupDN)
                          If objGroup.IsMember("LDAP://" & objUser.UserName) Then
                              objShell.Run "Migrate_PrintQs.vbs"
                          EndIf  
Alternatively, the script can be run from a Command Line (`cscript Migrate-PrintQs.vbs`).  
The script's Remove or Affix feature can be invoked either through Command Line switches when invoking the script (mostly used in interactive cases) or through specific formatting of mappings with the mappings file (mostly used via a Logon Script).  

# Script usage  
### Command Line switches  
* FileName.csv  
* /Affix:  
* /Remove:  
* /RemoveAllPrinters  
* /CheckGroupMembership  
* /CheckGroupMembership:<CustomGroupName>  

### Command Line examples  
`Migrate_PrintQs.vbs PrintMigTable.csv`  
Migrate current Print Queues based on the information inside specified 'PrintMigTable.csv' file. This file is to be posted on the Network Share specified in the 'PrintQMappingsRepo' variable  

`Migrate_PrintQs.vbs /Affix:\\ContosoNewPrtSrv\NewPrintQueueName`  
Add a mapping to '\\ContosoNewPrtSrv\NewPrintQueueName' if none already exists  

`Migrate_PrintQs.vbs /Remove:\\ContosoOldPrtSrv\OldPrintQueueName`  
Remove any mapping to '\\ContosoOldPrtSrv\OldPrintQueueName' if any exists  

`Migrate_PrintQs.vbs /RemoveAllPrinters`  
Remove all of user's printers  

`Migrate_PrintQs.vbs /CheckGroupMembership`  
Tell the script to act as if it is run within the Logon Script, meaning that the Mappings table to use is the default computed one for the user's devised Department.  

`Migrate_PrintQs.vbs /CheckGroupMembership:PrintMigUsers`  
Same as above but for special cases where the user does not comply to the Department OU = Group prefix = Mappings CSV file prefix paradigm. The migration will take place based on the Mappings table from the user's Department OU  

### Migration action examples via Mappings file  
     \\ContosoOldPrtSrv1\OldPrtQ1,\\ContosoNewPrtSrv1\NewPrtQ1
     \\ContosoOldPrtSrv1\OldPrtQ2,\\ContosoNewPrtSrv1\NewPrtQ2
     \\ContosoOldPrtSrv1\OldPrtQ3,\\ContosoNewPrtSrv1\NewPrtQ3
     \\ContosoOldPrtSrv1\OldPrtQ4,
     \\ContosoOldPrtSrv1\OldPrtQ5,DELETE
     \\ContosoNewPrtSrv1\NewGrpPrtQ,INSTALL

Which respectively will carry out the following actions:  
Replace if found '\\ContosoOldPrtSrv1\OldPrtQ1' by '\\ContosoNewPrtSrv1\NewPrtQ1'  
Replace if found '\\ContosoOldPrtSrv1\OldPrtQ2' by '\\ContosoNewPrtSrv1\NewPrtQ2'  
Replace if found '\\ContosoOldPrtSrv1\OldPrtQ3' by '\\ContosoNewPrtSrv1\NewPrtQ3'  
Remove '\\ContosoOldPrtSrv1\OldPrtQ4' if found  
Remove '\\ContosoOldPrtSrv1\OldPrtQ5' if found  
Add '\\ContosoNewPrtSrv1\NewGrpPrtQ' if not found  

# Script configuration  
There are 5 configurable variables (see lines 149 to 153 in the actual script) which need to be set by IT Administrator prior to using the present Script:  
* Variable **DeptsOU** contains the parent node OU in the form "OU=Departments" for instance where all departments are residing.  
* Variable **PrintServersOU** contains the OU where the Print Servers involved in the migration are located. Specifying this allows for faster (narrower) LDAP searches.  
* Variable **PrintMigGroupsOU** contains the OU where the different printer migrations Groups are residing. Specifying this allows for faster (narrower) LDAP searches.  
          Printer Migrations Groups for each BU/Department/OU are expected to match the pattern "_[SubOUNameInDeptsOUVar]-PrinterMigration_".  
          So for instance for HR, the script expects an 'HR' sub-OU inside **DeptsOU** above and the AD Group containing HR users which can migrate to be named "_HR-PrinterMigration_" and reside in AD inside the OU specified through this **PrintMigGroupsOU** variable.  
* Variable **PrintQMappingsRepo** contains the UNC location of where the mapping file(s) reside. (Ensure NTFS Security and Share permissions are set for 'Everyone' to READ).  
          Printer Mappings files to be posted here are expected to match string pattern "_<SubOUNameIn{DeptsOU}Var>-PrintQMig.csv_".  
          So for instance again for HR, the script expects an 'HR' OU inside **DeptsOU** above and the file containing the mappings for HR must be called "_HR-PrintQMig.csv_" and obviously must be present on the Network Share specified in this **PrintQMappingsRepo**.  
* Variable **PrintQMigrationLogs** contains the UNC location of where the user Print Queue migrations logs are to be created. (Ensure NTFS Security and Share permissions are set for 'Everyone' to WRITE).  


Note: The behaviour for user's Default Printer is that if, for whatever reason, no new Default Printer can be set in its place, it will never be removed.  
