**Install node js on Window**

 1. Method 1: Go to
    [https://nodejs.org/en/download/](https://nodejs.org/en/download/)
    to download the lastest version of Node js or the prevous versions
    at the link
    [https://nodejs.org/en/download/releases/](https://nodejs.org/en/download/releases/)
    
 2. Method 2: Install Node via NVM (Node Version Manager)
    [https://danielarancibia.wordpress.com/2017/03/28/install-or-upgrade-nodejs-with-nvm-for-windows/](https://danielarancibia.wordpress.com/2017/03/28/install-or-upgrade-nodejs-with-nvm-for-windows/)


**Uninstall completely from Windows**
1.  Take a deep breath.
    
2.  Run  `npm cache clean --force`
    
3.  Uninstall from Programs & Features with the uninstaller.
    
4.  Reboot (or you probably can get away with killing all node-related processes from Task Manager).
    
5.  Look for these folders and remove them (and their contents) if any still exist. Depending on the version you installed, UAC settings, and CPU architecture, these may or may not exist:
    
    -   `C:\Program Files (x86)\Nodejs`
    -   `C:\Program Files\Nodejs`
    -   `C:\Users\{User}\AppData\Roaming\npm`  (or  `%appdata%\npm`)
    -   `C:\Users\{User}\AppData\Roaming\npm-cache`  (or  `%appdata%\npm-cache`)
    -   `C:\Users\{User}\.npmrc`  (and possibly check for that without the  `.`  prefix too)
    -   `C:\Users\{User}\AppData\Local\Temp\npm-*`
6.  [Check your  `%PATH%`  environment variable](https://stackoverflow.com/questions/141344/how-to-check-if-directory-exists-in-path)  to ensure no references to  `Nodejs`  or  `npm`  exist.
    
7.  If it's  _still_  not uninstalled, type  `where node`  at the command prompt and you'll see where it resides -- delete that (and probably the parent directory) too.
    
8.  Reboot, for good measure.

[https://stackoverflow.com/questions/20711240/how-to-completely-remove-node-js-from-windows](https://stackoverflow.com/questions/20711240/how-to-completely-remove-node-js-from-windows)

**Using global modules on Window**

Add an environment variable called `NODE_PATH` and set it to `%USERPROFILE%\Application Data\npm\node_modules` (Windows XP), `%AppData%\npm\node_modules` (Windows 7/8/10), or wherever npm ends up installing the modules on your Windows flavor. To be done with it once and for all, add this as a System variable in the Advanced tab of the System Properties dialog (run `control.exe sysdm.cpl,System,3`).

[https://stackoverflow.com/questions/9587665/nodejs-cannot-find-installed-module-on-windows](https://stackoverflow.com/questions/9587665/nodejs-cannot-find-installed-module-on-windows)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzExNTcyOTcxLDM1NDUwMzUxNV19
-->