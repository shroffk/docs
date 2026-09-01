## EIC GNOME Desktop Configuration (OS & User)
****

## Overview

- This setup ensures a baseline, user-friendly GNOME environment with preferred defaults across the system.
- Installed the GNOME desktop and all required tools and extensions, including GNOME Tweaks, Extensions App, Alacarte and commonly used shell extensions (dash-to-dock, apps-menu, window-list, desktop-icons, etc):
  - The dash-to-dock extension provides a persistent application dock for easy access to favorite and running applications.
  - The application menu is now displayed by default, improving access to installed applications.
  - The desktop-icons extension adds right-click options such as “Open Terminal” and “Show Desktop in Files”.
  - The window-list improves window/task switching
- Applied system-wide default preferences:
  - Enabled GNOME Shell extensions
  - Set default favorite applications
  - Configured workspace layout and fixed number of workspaces
  - Enabled minimize / maximize / close buttons on all windows
  - Adjusted Nautilus icon size (default → standard)
  - Configured GNOME Terminal to launch in ~/ instead of ~/Desktop
  - Applied dconf locks for screensaver and notificationsEnabled GNOME Shell extensions.
  

## Scripts

- /eic/opt/installation/gnome-desktop/setup-gnome-os.sh
- /eic/opt/installation/gnome-desktop/setup-gnome-user.sh

## Ansible Roles

- https://github.com/eicorg/proxmox_eval/tree/main/ansible/roles/setup_gnome_os
- https://github.com/eicorg/proxmox_eval/tree/main/ansible/roles/add_user_to_gnome

 
## Reference (CAD Documentation)

- https://www.cadops.bnl.gov/Controls/doc/SA/RH96/RH96-Documents/doc-desktopGnome/gnome-configure-os.html  
- https://www.cadops.bnl.gov/Controls/doc/SA/RH96/RH96-Documents/doc-desktopGnome/gnome-configure-normaluser.html  

----

<img width="587" height="364" alt="image" src="https://github.com/user-attachments/assets/f325ca96-500d-41e5-9ebe-822da2a8593c" />

## Detail Steps  

**1. Install RPM Packages**

```bash
alacarte
gnome-tweaks.noarch
gnome-extensions-app
dconf-editor
dbus-x11

gnome-shell-extension-apps-menu.noarch
gnome-shell-extension-auto-move-windows.noarch
gnome-shell-extension-common.noarch
gnome-shell-extension-dash-to-dock.noarch
gnome-shell-extension-desktop-icons.noarch
gnome-shell-extension-drive-menu.noarch
gnome-shell-extension-launch-new-instance.noarch
gnome-shell-extension-native-window-placement.noarch
gnome-shell-extension-panel-favorites.noarch
gnome-shell-extension-places-menu.noarch
gnome-shell-extension-screenshot-window-sizer.noarch
gnome-shell-extension-systemMonitor.noarch
gnome-shell-extension-top-icons.noarch
gnome-shell-extension-updates-dialog.noarch
gnome-shell-extension-user-theme.noarch
gnome-shell-extension-window-list.noarch
gnome-shell-extension-windowsNavigator.noarch
gnome-shell-extension-workspace-indicator.noarch
```

**2. Copy/Update .desktop**

```bash
/bin/cp -pf /eic/opt/installation/gnome-desktop/logout.desktop /usr/share/applications/
/bin/cp -pf /eic/opt/installation/gnome-desktop/lock-screen.desktop /usr/share/applications/
/bin/cp -pf /eic/opt/installation/gnome-desktop/lock.png  /usr/share/pixmaps/

sed -i 's/NoDisplay=true/NoDisplay=false/' /usr/share/applications/libreoffice-startcenter.desktop
update-desktop-database
```
NOTE:
> In RHEL 9, LibreOffice Start Center is hidden by default (NoDisplay=true).  
> Setting NoDisplay=false makes it visible in the Applications menu and Dock.  


**3. Setup default system wide favorite apps, screensaver, icon view size, workspace primary, disable notification, desktop-icons**

```bash
/bin/cp -pf /eic/opt/installation/gnome-desktop/system-default  /etc/dconf/db/local.d/system-default

/bin/cp -pf /eic/opt/installation/gnome-desktop/locks/screensaver /etc/dconf/db/local.d/locks/screensaver
/bin/cp -pf /eic/opt/installation/gnome-desktop/locks/notification /etc/dconf/db/local.d/locks/notification
```

**4. Enable GNOME Extensions**

```bash
/usr/bin/dbus-launch gsettings set org.gnome.shell enabled-extensions "['dash-to-dock@gnome-shell-extensions.gcampax.github.com', 'apps-menu@gnome-shell-extensions.gcampax.github.com', 'launch-new-instance@gnome-shell-extensions.gcampax.github.com', 'places-menu@gnome-shell-extensions.gcampax.github.com', 'window-list@gnome-shell-extensions.gcampax.github.com', 'desktop-icons@gnome-shell-extensions.gcampax.github.com']"
```

**5. Set Favorite Applications**

```bash
/usr/bin/dbus-launch gsettings set org.gnome.shell favorite-apps "['firefox.desktop', 'libreoffice-startcenter.desktop', 'org.gnome.Terminal.desktop', 'org.gnome.tweaks.desktop', 'org.gnome.Extensions.desktop', 'gnome-control-center.desktop', 'lock-screen.desktop', 'logout.desktop']"
```

**6. Configure GNOME Settings**

```bash
# By default ICON size for Nautilus has been set to large. Change to small
/usr/bin/dbus-launch gsettings set org.gnome.nautilus.icon-view default-zoom-level standard

# By default windows list group mode is never. Change to auto
/usr/bin/dbus-launch gsettings set org.gnome.shell.extensions.window-list grouping-mode auto

# By default workspaces-only-on-primary is true. Change to false
/usr/bin/dbus-launch gsettings set org.gnome.mutter workspaces-only-on-primary false

# Use fixed number workspaces
/usr/bin/dbus-launch gsettings set org.gnome.mutter dynamic-workspaces false

# By default no minimize or maximize buttons on the open application
/usr/bin/dbus-launch gsettings set org.gnome.desktop.wm.preferences button-layout ":minimize,maximize,close"

# Set some default workspaces
/usr/bin/dbus-launch gsettings set org.gnome.desktop.wm.preferences workspace-names "['Workspace 1', 'Workspace 2', 'Workspace 3', 'Workspace 4']"

# Set screen saver default time
/usr/bin/dbus-launch gsettings set org.gnome.desktop.session idle-delay 900

# Set use-command for gnome-terminal so that it opens as home folder (see Appendix)
/usr/bin/dbus-launch gsettings set org.gnome.Terminal.Legacy.Profile:/b1dcc9dd-5262-4d8d-a863-c897e6d979b9/ use-custom-command true
/usr/bin/dbus-launch gsettings set org.gnome.Terminal.Legacy.Profile:/b1dcc9dd-5262-4d8d-a863-c897e6d979b9/ custom-command '/bin/tcsh'

# Update 
dconf update 
```
  
----

**Appendix - GNOME Terminal Default Directory**

**Problem:**
- In GNOME Desktop, when user right-click on the desktop and select "Open in Terminal", a new gnome-terminal process starts default in ~/Desktop directory. We would like to have it open in ~/ directory.
Use GUI (the safest way)
- https://bugs.launchpad.net/ubuntu/+source/gnome-terminal/+bug/1587154
- https://access.redhat.com/support/cases/#/case/03194924

**Solution**
- The solution is to set two values in the legacy gnome terminal.

- To do it via GUI (the safest way)
  - In the Terminal window, click "Edit" -> "Preferences".
  - Under "Command" tab,
    - Check "Run a custom command instead of my shell"
    - Custom command: /bin/tcsh
    - Close and open a new Terminal. No need to logout.  

      <img width="439" height="266" alt="image" src="https://github.com/user-attachments/assets/78a61f4b-7929-428b-9ec5-75dbda5b3c01" />

- To do it via "dconf-editor"
  - On a terminal, run "dconf-editor"
  - Browse to org/gnome/terminal/legacy/profiles:/:b1dcc9dd-5262-4d8d-a863-c897e6d979b9
  - Custom command: /bin/tcsh
  - use-custom-command: ON  

    <img width="918" height="174" alt="image" src="https://github.com/user-attachments/assets/bd336189-052a-46fc-a320-c55447e834d0" />


- Do do it via command line
  ```bash
  gsettings set org.gnome.Terminal.Legacy.Profile:/b1dcc9dd-5262-4d8d-a863-c897e6d979b9/ use-custom-command true
  gsettings set org.gnome.Terminal.Legacy.Profile:/b1dcc9dd-5262-4d8d-a863-c897e6d979b9/ custom-command '/bin/tcsh'
  ```

- To verify
  ```bash
  gsettings get org.gnome.Terminal.Legacy.Profile:/b1dcc9dd-5262-4d8d-a863-c897e6d979b9/ use-custom-command
  gsettings get org.gnome.Terminal.Legacy.Profile:/b1dcc9dd-5262-4d8d-a863-c897e6d979b9/ custom-command
  ```

 ----
