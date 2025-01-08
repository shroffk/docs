## GitHub IOC Repository Structure

- On GitHub, under the `eicorg` account, the IOCs are hosted under the controls team. Here is the path: https://github.com/orgs/eicorg/teams/controls/repositories.
- Any IOC repository name has the pattern: `ioc-<system name>`.

- Before creating/pushing any IOC repository, ensure that you have the following directory structure:
```
.
└──ioc-<system name>
    ├── <system name>App
    ├── iocBoot
    ├── Makefile
    └── .gitignore
```

- Put all source code for the IOC inside `<system name>App` and startup files with OPIs under `iocBoot`.

- iocBoot can have multiple sub-directories each corresponding to each user group.

- A later version of iocBoot directory inside the IOC repository (on GitHub) will contain a version of cookiecutter (Jinja) compatible `st.cmd` and `config` files (for manage-iocs) so that it can be deployed using web/GUI for each user.


- The content of the `.gitignore` will look like the following one, so that all auto-generated files are excluded from the repository:
```
# Install directories
/bin/
/cfg/
/db/
/dbd/
/html/
/include/
/lib/
/templates/

# Local configuration files
/configure/*.local

# iocBoot generated files
/iocBoot/*ioc*/cdCommands
/iocBoot/*ioc*/dllPath.bat
/iocBoot/*ioc*/envPaths
/iocBoot/*ioc*/relPaths.sh

# Build directories
O.*/

# Common files created by other tools
/QtC-*
/.vscode/
*.orig
*.log
.*.swp
.DS_Store
```

## EPICS IOC Deployment Plan

- EPICS IOC apps, iocBoot and OPIs are hosted on GitHub. They are released and deployed through GitHub actions or user interface (web or GUI) only, so that we have all releases synchronized with GitHub. Until GitHub actions or user interface is set up, we need to do it manually.

- On the EIC VMs, the central NFS directory for deployment is under `/eic/release/epics`. Under this NFS mounted central directory, we have

```
.
└── epics
    ├── ioc
    ├── iocBoot
    └── opi
```

- Inside, the `iocBoot` and `opi` directories, new directories can be created that reflect the tree structure of the users or group.

- For any EPICS IOC app development, the users might want to save the local source repository under `/eic/source/epics`. For each IOC app, i) IOC app source code, ii) iocBoot and iii) OPI files can be kept under one repository. However, the most important thing is to keep the local repository synchronized with the GitHub repository.

- The IOCs are be started/stopped/monitored using the `manage-iocs` utility or an user interface (on top of manage-iocs).

- Because we maintain an independent cental `iocBoot` directory (which is not next to the apps directory) as shown in the above diagram, we need to copy the auto-generated `db` and `dbd` directories inside `iocBoot/<ioc name>` directory. Also, we need to set the `$TOP` variable inside `envPaths` to point to the `iocBoot/<ioc name>` directory. Similarly, we need to copy the binary IOC executable to a central directory (here `ioc`). All of these should be made part of the deployment script. This way each user will have their own `iocBoot/<ioc name>` directory, but they will be using the same IOC executable. Thus each user/group will have the capability to modify the database file on the fly to fit their needs. `<ioc name >` in the `iocBoot/<ioc name>` should contain a unique name like `<user_name-hardware_name>`.

- The `iocBoot/<ioc name>` directory will contain a `config` file for the `manage-iocs` utility.

- Thus the build script (or GitHub action) will do the following:
	- Run make for the source IOC code after downloading it from GitHub.
	- Copy IOC executable to a central location
	- Copy `iocBoot` with `db` and `dbd` directories to a central location separated by the user name.
	- Create a `config` file to be used with the manage-iocs utility under the `iocBoot/<ioc name>` directory.
	- Copy the OPI to a central location separated by the user name
	- Now you should be able to install/start/stop the IOC using manage-iocs.

- A later version of iocBoot directory inside the IOC repository (on GitHub) will contain a version of cookiecutter (Jinja) compatible `st.cmd` and `config` files (for manage-iocs) so that it can be deployed using web/GUI for each user.
