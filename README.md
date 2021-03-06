# PSMenu

PSMenu started out from my attempts to find a good way to present a menu of options to administrators consuming my PowerShell modules and functions.  
There are some good options out there, especially using $host.UI.PromptForChoice(), which I do use in a variety of situations. 
PromptForChoice, however, only provides part of what is required for a menu system.  PSMenu is envisioned to provide the following features:

* Flexible Presentation of User Choices (the Show-Menu function currently provides one option for display of choices but can be expanded.  I plan to add PromptForChoice as an option in one of the next revisions.)
* Menu Hierachies and/or Nested Menus with dynamic inclusion of Child menus in the choices list, if desired.
* Dynamically generated menus from any array of objects
* Separation of menu definition from menu display and user choice execution.  Menus are defined as a pscustomobject.

## How it Works
1. Create a menu definition object (a pscustomobject), with the following properties:
    - GUID: GUID or other Unique Identifier
    - Title: Descriptive String which will be displayed to the user as the Menu Title when the menu is invoked.
    - Initialization: Any arbitrary string to be converted to a scriptblock to be run when the menu is invoked, stored as a string to be convereted to scriptblock dynamically at menu invocation.  
    - Choices: An array of objects with properties "choice" and "command".  Choice is a string which describes the option to the user.  Command is an arbitrary string to be converted to scriptblock and run when the menu choice is selected.  Stored as a string to be converted to a scriptblock dynamically at menu invocation. 
    - ParentGUID: used for determining menu location in a hierarchy and affects display / navigation options presented to the user.  
    - DefaultDisplayMode (Future option, not currently implemented, to select a text based template, PromptForChoice, or other yet to be determined options)
Alternatively, try using New-DynamicMenuDefinition, like this: $menudefinition = New-DynamicMenuDefinition -Title "Show properties of the selected file" -Choices (ls | select-object -expandproperty fullname) -command "Get-Item" -ChoiceAsCommandParameter 
2. Run Invoke-Menu -menudefinition $menudefinition
    - Invoke-menu calls function New-MenuScriptblock to dynamically create a scriptblock which is then executed.  The scriptblock created by New-MenuScriptblock includes the Show-Menu function which handles the display and input of user choices. User choice is returned to the scriptblock and the appropriate arbitrary code in the scriptblock corresponding to the user choice is executed. 
    
## Development Plans

1. Make inclusion of child menus as a choice optional via parameter
2. Improve the Menu Hierarchy logic and functions
3. Make screen clearing optional via parameter
4. Modify New-DynamicMenuDefinition to make command pipeline possible (place choice token/identifier arbitrarily within the command string)
5. Add Import and Export functions to import and export menu definitions to JSON

## Note concerning dynamic scriptblocks
* To be successful, you will need to be aware of the rules variable scoping in relation to script blocks created by these functions.
* When creating dynamic scriptblocks from strings you will need to control when variable expansion occurs in strings to achieve your intended effect.  Various combinations of single(') and double(") quotes and the backtick character (`) can usually help you achieve your goal.  

## Example
cd c:\
md test

        Directory: C:\


    Mode                LastWriteTime     Length Name                                                                                                                                                                      
    ----                -------------     ------ ----                                                                                                                                                                      
    d----        2015-05-29  11:33 AM            test         

1..3 | foreach {New-Item -Name "document$_.txt" -ItemType file}

        Directory: C:\test


    Mode                LastWriteTime     Length Name                                                                                                                                                                      
    ----                -------------     ------ ----                                                                                                                                                                      
    -a---        2015-05-29  11:36 AM          0 document1.txt                                                                                                                                                             
    -a---        2015-05-29  11:36 AM          0 document2.txt                                                                                                                                                             
    -a---        2015-05-29  11:36 AM          0 document3.txt     

$menudefinition = New-DynamicMenuDefinition -Title "Select a file to delete" -Choices (ls | select -ExpandProperty fullname) -command "Remove-Item" -ChoiceAsCommandParameter

$menudefinition

    GUID           : e72f2d4e-7eeb-4512-abee-fad950d94ca1
    Title          : Select a file to delete
    Initialization : 
    Choices        : {@{choice=C:\test\document1.txt; command=Remove-Item C:\test\document1.txt}, @{choice=C:\test\document2.txt; command=Remove-Item C:\test\document2.txt}, @{choice=C:\test\document3.txt; 
                     command=Remove-Item C:\test\document3.txt}}
    ParentGUID     : 

Invoke-Menu -menudefinition $menudefinition

    Current Menu: Select a file to delete
    ======================================================================================================================

	    1 C:\test\document1.txt
	    2 C:\test\document2.txt
	    3 C:\test\document3.txt

    ======================================================================================================================

    Enter your selection or 'Q' to exit this menu
    :: 2

ls

        Directory: C:\test


    Mode                LastWriteTime     Length Name                                                                                                                                                                      
    ----                -------------     ------ ----                                                                                                                                                                      
    -a---        2015-05-29  11:36 AM          0 document1.txt                                                                                                                                                             
    -a---        2015-05-29  11:36 AM          0 document3.txt        

## License

PSMenu is released under the MIT License
