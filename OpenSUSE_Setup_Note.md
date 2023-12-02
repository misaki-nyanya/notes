# OpenSUSE Setup Note

> Recommend not to change software package in installation wizard.
> Especially, if you choose rpm-ndb, system will uncheck rpm automatically.
> But as you may not install ndb backend, the system will not be able to 
> install any package after the completion of installation wizard.

##Steps after installation.
1. Make the following folders load as tmpfs and disable swap.

        tmpfs  /var/tmp   tmpfs  defaults,noatime,mode=1777  0  0
        tmpfs  /var/log   tmpfs  defaults,noatime,mode=0755  0  0
        tmpfs  /tmp       tmpfs  defaults,noatime,mode=1777  0  0

2. Add Chinese mirror, Packman and VLC
    Refer to OpenSUSE official site.

3. Config Firefox.
    1. Access about:config
        * set using memory cache instead of disk cache
        * dom.w3c_touch_events.enabled = 1 to enable touch screen scroll.
	2. Edit system file to enbale touch screen scroll
	    * $ sudo nano /etc/security/pam_env.conf
		* Add line "MOZ_USE_XINPUT2 DEFAULT=1"

4. Make USB available. By default USB will be stopped by TLP.
	1. Find the AMD USB controllers.

    	       $ sudo lspci
                03:00.4 USB controller: Realtek Semiconductor Co., Ltd. RTL811x EHCI host controller (rev 0e)
        		06:00.3 USB controller: Advanced Micro Devices, Inc. [AMD] Raven USB 3.1
        		06:00.4 USB controller: Advanced Micro Devices, Inc. [AMD] Raven USB 3.1

    2. Add them to the RUNTIME_PM_BLACKLIST in /etc/tlp.conf

            RUNTIME_PM_BLACKLIST="03:00.4 06:00.3 06:00.4"

	3. Reboot - USB works on battery whilst TLP is still functional.

5. Make home folder and command line en_US.

    add "export LANG=en_US" to **.bashrc** or **.profile**

            $ export LANG=en_US
            $ xdg-user-dirs-gtk-update

    check **~/.config/user-dirs.dirs**

            XDG_DESKTOP_DIR="$HOME/Desktop"
            XDG_DOWNLOAD_DIR="$HOME/Downloads"
            XDG_TEMPLATES_DIR="$HOME/Templates"
            XDG_PUBLICSHARE_DIR="$HOME/Public"
            XDG_DOCUMENTS_DIR="$HOME/Documents"
            XDG_MUSIC_DIR="$HOME/Music"
            XDG_PICTURES_DIR="$HOME/Pictures"
            XDG_VIDEOS_DIR="$HOME/Videos"


6. Install ffmpeg to enable media in firefox.

    Add packman or VLC repo.

	       $ sudo zypper install ffmpeg-4

	(libavcodec56, libavcodec57, libavformat56, libavformat57, libavdevice56, libavdevice57 shall also be installed)

7. Install amd driver
	
    Install amdgpu(need amd repo) or from AMD official site installer, amdgpu-installer.

            $ sudo zypper install amdgpu

8. Stop beep sound.

    Add following line in **.bashrc** or **.profile**
	xset b off

9. Enable launching GUI application from Terminal.

    Maybe the hostname is not set static

    		$ sudo hostnamectl set-hostname <your host name>
    		$ hostnamectl   #check again


10. Enable Natural scroll of touchpad.

    	$ xinput list		      # touchpad is 14
    	$ xinput --list-props 14      # natural scroll is 338
    	$ xinput set-prop 14 338 1    # set natural scroll to 1 
	**Gestures** and **libinput-gestures** are good but they are not provided in default repos.
	may try to compile them by yourself.

11. Problem *Authorization required, but no authorization protocol specified*.

    Add following in **.bashrc** or **.profile** may solve this for GUI applications.

        	xhost +si:localuser:$USER &
    > "The slightly more detailed answer is those programs require X windows features but there's no configuration available in the root account for that to happen."



    Here is an answer found on [StackExchange](https://unix.stackexchange.com/questions/209746/how-to-resolve-no-protocol-specified-for-su-user)

    > Applications need a "magic cookie" (secret token) in order to talk to the X11 server so that other processes on the system running under other users cannot intrude on your display, create windows, and snoop your keystrokes. The other system user does not have access to this magic cookie because the permissions are set so that it is only accessible to the user who started the desktop environment (which is as it should be).

    > Try this, running as your original user, to copy the X11 cookie to the other account:

                $ su - <otheruser> -c "unset XAUTHORITY; xauth add $(xauth list)"

    > then run your application. You may also need to unset XAUTHORITY in that shell too. That command extracts the magic cookie (xauth list) from your main user and adds it (xauth add) to where the other user can get it.

    > You'll have to redo it each time after you log out and log in because a brand new magic cookie gets generated each time. That's normal, it's part of the security model.
    
    > The man page for xauth specifically identifies the "MIT-MAGIC-COOKIE-1" portion of the entries as the "protocolname". 


    Another answer on the same page:

    > There might be several ways to run the xauth commands, for example, you might be using 'sudo', but that might lose or change environment variables. The following environment variables need to be preserved: DISPLAY and XAUTHORITY. To test if that is the case you could run 'echo $XAUTHORITY' in the same way you run your commands, but make sure you aren't expanding the environment variables before you run those commands. For example, try: sudo bash -c 'echo "$XAUTHORITY"' to see what XAUTHORITY really is after you run your sudo (if it disappears you might need to add something to your sudoers file, see elsewhere).

    > Eventually, run the following command as the user that you want to get access with, on the server:

                xauth info

    > This will show the 'Authority file' that will be used (/root/.Xauthority by default, for root, or something like /home/theuser/.Xauthority). If it shows the correct .Xauthority file then you don't have to worry about the XAUTHORITY environment variable actually (actually, I wouldn't know when it wouldn't, except if you want to manipulate a non-standard place of that file).

    > Remove that file (if it even exists):

                mv /root/.Xauthority /root/.Xauthority.bak

    > In the above command, replace /root/.Xauthority with the correct XAUTHORITY file for your case of course.

    > Recreate it, but empty (this is needed for a lot of commands):

                touch /root/.Xauthority

    > At this point you'll get the No protocol specified error, even if you got Invalid MIT-MAGIC-COOKIE-1 before. Find the authority file that the X server is using at the moment:

                ps aux | grep Xorg

    > This should show something like:

                root      1153  0.0  1.0 149560 44464 tty7     Ss+  dec02   0:00 /usr/lib/xorg/Xorg -nolisten tcp -auth /var/run/sddm/{ef18c483-7891-4e82-80ef-2c8f9bd79711} -background none -noreset -displayfd 17 vt7

    > The file name after -auth is what you need in the next command. Run this as root:

                sudo xauth -f '/var/run/sddm/{ef18c483-7891-4e82-80ef-2c8f9bd79711}' list

    > That lists a 32 digit hexadecimal key. For example the output could be:

                hostname/unix:0  MIT-MAGIC-COOKIE-1  c0eaf749aa252101a0f57d5087089db7

    > Use that to generate your .Xauthority file (as user who needs to login again):

                xauth add $DISPLAY MIT-MAGIC-COOKIE-1 c0eaf749aa252101a0f57d5087089db7

    > replace 'c0eaf749aa252101a0f57d5087089db7' with what was returned by the list command for you. Now your .Xauthority should be size 51 bytes and you can connect to the X server (again).

    > PS If you start Xorg by running /usr/bin/startx, like me; then you might see something like:

                root        1652  0.0  0.0  12788  5792 ?        Ss   jan20   0:00 login -- carlo
                carlo       1834  0.0  0.0   8140  3468 tty1     Ss   jan20   0:00  \_ -bash
                carlo       1887  0.0  0.0   7404  3236 tty1     S+   jan20   0:00      \_ /bin/sh /usr/bin/startx
                carlo       1905  0.0  0.0   3912   828 tty1     S+   jan20   0:00          \_ xinit /home/carlo/.xinitrc -- /etc/X11/xinit/xserverrc :0 vt1 -keeptty -auth /tmp/serverauth.WWPpq4OSlA
                root        1906  1.2  0.7 25576848 235104 tty1  Sl   jan20 207:56              \_ /usr/lib/Xorg -nolisten tcp :0 vt1 -keeptty -auth /tmp/serverauth.WWPpq4OSlA
                carlo       1917  0.0  0.0 143408 10884 tty1     Sl   jan20   0:00              \_ startplasma-x11

    > And the /tmp/serverauth.WWPpq4OSlA was deleted. See /usr/bin/startx script for how this works:

                mcookie=`/usr/bin/mcookie`
                xserverauthfile=`mktemp -p /tmp serverauth.XXXXXXXXXX`
                trap "rm -f '$xserverauthfile'" HUP INT QUIT ILL TRAP KILL BUS TERM
                xauth -q -f "$xserverauthfile" << EOF
                add :$dummy . $mcookie
                EOF

    > as well as

                xauth -q << EOF
                add $displayname . $mcookie
                EOF

    > adding the random cookie to the users .Xauthority file.

    > In this case the cookie has completely vanished thus. The only place where you can get it back from is the memory (RAM) of the Xorg process; but I am too lazy to figure out how to recover that. Just restarting X should regenerate your .Xauthority file with a new cookie and also restart the server with that same cookie, of course.

    Simply export XAUTHORITY=~/.Xauthority may also works...


