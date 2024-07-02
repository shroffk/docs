## Create Standalone Machine - Default Controls Server Management - Server Action Layout

We wish to create server action layout which can be generic and also allows users to easily customize for any special servers.

Here is the layout of the structure.

Files to perform the Actions are in:
	- templates: /home/cfsd/sysadmin/templates/Linux/StandAlone/ActionScripts 
	- During OS setup, they will be copied over to /home/here/cadLocal by the script makeLocalDirs.runasroot
	- note:  You need to update the action files
		- ActionScripts contains camera name...need to update to your camera name (will move to database later)
		- ActionScripts contains aravis path...need to update to your current path

Controlling the ActionScripts - syndiDisplay
- started by running  "sudo /usr/controls/scripts/easyControl"
	- This will launch the syndi display - running it as the gatling user
		- /usr/controls/scripts/syndiViewerTest
	- Supporting files and scripts for the syndiDisplay are:
     	- /operations/app_store/syndi/repository/ActionScripts.xml       

The scripts that do the actual work are (where ACTION may be [ Check | Start | Stop | Restart | List | Help ]
- AllServersWrapper ACTION  --> For Stop and Restart, a user must confirm the action
	- AllServers ACTION  
		- BasicServers ACTION
		- SystemSpecificServers ACTION
			- PerformSystem $system ACTION
				- PerformProcess.pl $system ACTION
Note:
	- The BasicServers script and SystemSpecificServers script look for files that contain the list of basic servers and system specific servers (see below for file explication)
	- The BasicServers script and SystemSpecificServers script may be called independently, passing a valid value for ACTION
	- The PerformSystem script could be removed and the BasicServers and SystemSpecificServers script could call PerformProcess.pl directly (the functionality of looping through multiple processes/servers in the same system, initially in PerformSystem) has been moved into the PerformProcess.pl script)
	- All of the work is done in the PerformProcess.pl script             

Explanation of ACTIONS:
1. Check: Determines if process is running using a pgrep -lf.
2. Start: Checks to make sure process is not already running, if it is, will not Start, otherwise starts process. 
	- Output of process is redirected to a /tmp/PROCESS_NAME.out file. If the file already exists, it is moved to a /tmp/PROCESS_NAME.out.DATE file
3. Stop: Checks to make sure process is running.  If it is, it stops it
4. Restart: Stop - Start

Directory structure and support files for the ActionScripts:
	- The scripts above should be placed in a $ACTION_SCRIPT_BASE directory
	- This environment variable is defined in /home/here/cadLocal/ActionScripts/ENV.
	- Default is /home/here/cadLocal/ActionScripts
	- The directories should look like
        $ACTION_SCRIPT_BASE
        $ACTION_SCRIPT_BASE/servers
        $ACTION_SCRIPT_BASE/system
        $ACTION_SCRIPT_BASE/system/$SYSTEM_NAME
        $ACTION_SCRIPT_BASE/system/$SYSTEM_NAME/[list, pre, environment]
- The files needed to drive the ActionScripts system are
	$ACTION_SCRIPT_BASE/servers/basicList - list of all basic processes/servers to be handled by an ACTION
	$ACTION_SCRIPT_BASE/servers/systemSpecificList - list of all system specific processes/servers to be handled by an ACTION
- A system listed in the basicList or systemSpecificList is automatically handled ('generic') as follows:
	- The name in the list exactly matches a single process/server that resides in /usr/controls/bin
If a system does not follow the 'generic' template:
	- A directory (see above - $ACTION_SCRIPT_BASE/system/$SYSTEM_NAME) for the system needs to be created and populated with files that expound the system     
	- A system 'list' file needs to be created:
		- $ACTION_SCRIPT_BASE/system/$SYSTEM/list
		The list file contents explain the system
			- Each process that makes up the system is entered on a line
			- Each line in the file contains 6 values, comma-separated
		(Comments are from the PerformProcess.pl file)
		1) SystemName
		2) The path to the program
		3) Start or Check String (in quotes) - A specific string used to start the server or check if the server is running (for distinguishing servers with same name, different instance)
		4) Alternate Check String (in quotes) - in case the start/check string in 3 runs a script, then the check may really need to be customized
		5) Error File Specification or extra arguments to start command - (This may be NONE)
		6) The Server Name (to distinguish from other servers that belong to the same system
		7) A specialized stop command - (This may be NONE)
	A system 'environment' file MAY be created
		- $ACTION_SCRIPT_BASE/system/$locSystemName/environment
		- This file is sourced prior to starting each process on an associated line        
	A system 'pre' or 'post' file MAY be created
		- $ACTION_SCRIPT_BASE/system/$locSystemName/[pre,post]
		- The pre file would be executed before the system is started.  The post file would be executed after the system is started. It must be guaranteed that the file could be started multiple times without causing damage.
		- If the ControlsNameServer were to be started, and there were a post file such as $ACTION_SCRIPT_BASE/system/cns/post, the file could be executed to fill the cns after the ControlsNameServer were started.
		- Similarly, if the camera system needed to be started, by cns entries needed to be added prior to starting it, the $ACTION_SCRIPT_BASE/system/camera/pre file could be executed prior to the ebicman processes being started.  

Example:
	- Include the StartUp process in the basicList file - no system directory for special handling
	- Check - looks for /usr/controls/bin/StartUp to be running
	- Start - first checks - if running, will not start, otherwise, starts /usr/controls/bin/StartUp process with stdout and stderr redirected to /tmp/StartUp.out
	- Stop - first checks - if running, stops process with a pkill -f /usr/controls/bin/StartUp
	- Restart - Stop - then Start












