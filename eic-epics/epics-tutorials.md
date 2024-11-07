# EPICS Tutorials

## Creating a demo/test ioc

- The following instructions explain how to create a test IOC on any EIC VM where the epics-base is already installed.

```
mkdir -p testIoc
cd testIoc
makeBaseApp.pl -t example testIoc
makeBaseApp.pl -i -t example testIoc
make
cd iocBoot/ioctestIoc
chmod u+x st.cmd
./st.cmd
# Next few lines are just output on epics-shell. You can ignore and jump to the prompt (epics>)
#!../../bin/darwin-x86/testIoc
< envPaths
epicsEnvSet("IOC","ioctestIoc")
epicsEnvSet("TOP","/Users/maradona/EPICS/TEST/testIoc")
epicsEnvSet("EPICS_BASE","/Users/maradona/EPICS/epics-base")
cd "/Users/maradona/EPICS/TEST/testIoc"
## Register all support components
dbLoadDatabase "dbd/testIoc.dbd"
testIoc_registerRecordDeviceDriver pdbbase
## Load record instances dbLoadTemplate "db/user.substitutions"
dbLoadRecords "db/testIocVersion.db", "user=junkes"
dbLoadRecords "db/dbSubExample.db", "user=junkes"
#var mySubDebug 1
#traceIocInit
cd "/Users/maradona/EPICS/TEST/testIoc/iocBoot/ioctestIoc"
iocInit
Starting iocInit
############################################################################
## EPICS R7.0.1.2-DEV
## EPICS Base built Mar 8 2018
############################################################################
iocRun: All initialization complete
2018-03-09T13:07:02.475 Using dynamically assigned TCP port 52908.
## Start any sequence programs
#seq sncExample, "user=maradona"
epics> dbl
maradona:circle:tick
maradona:compressExample
maradona:line:b
maradona:aiExample
maradona:aiExample1
maradona:ai1
maradona:aiExample2
... etc. ...
epics>
```
Now in another terminal, one can try command line tools like
```
caget, caput, camonitor, cainfo (Channel Access)
pvget, pvput, pvlist, eget, ... (PVAccess)
```

- Note: In general, the syntax to start IOC is: `<ioc executable>  <cmd file name>`. The `<ioc executable>` is placed under the `<module name>/bin` directory. 
In the above example, the path to the ioc executable is already given as the comment in the very first line of `st.cmd` file. So, you could just do/run as `./st.cmd`.

- On the epics-shell, type `dbl` to see a list of available process variables.

- Run css phoebus from the terminal and visualize your PV on the `css phobus` to monitor the data. 


## Simulated StreamDevice

- This tutorial explains how to use a python script as a simulated StreamDevice and then one can use the provided example IOC to interact with the device.

- On any EIC VM, go to the directory `/eic/opt/epics-iocs/streamDevSim`.

- The file `run_sim_dev.py` can be considered a simulated stream device hardware. You can start it using `python run_sim_dev.py` and it will start the server at `localhost:5000`. Once started, you can try to connect to the port `localhost:5000`, and for any read/write request (writing anything) sent to this port, it will return a temperature and humidity value.

- The IOC app creates an IOC that uses `calc`, `asyn` and 'stream` EPICS modules to talk to the simulated StreamDevice synchronously/asynchronously.

- Once you have IOC executable, run the simulated stream device (HW) from one terminal using `python run_sim_dev.py` and from another terminal start your IOC and look at the corresponding PVs.

- Starting the IOC:
  - `cd streamDevSim/exIOC/iocBoot/iocexIOC`.
  - chmod +x st.cmd (if it is not executable arleady)
  - `./st.cmd`
  - Print list of PVs (from IOC shell): `dbl`
  - Check PV value (from IOC shell): `dbgf simDev:humi`
  - Example to monitor PVs (from another terminal): `pvmonitor simDev:humi`

- Use Phoebus to monitor the PVs.

