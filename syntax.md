---
layout: page
title: Hall Of Syntax
permalink: /syntax/
---

It's the place where I write down somewhat useful syntax snippets. Instead of keeping my local copy I decided to make it public. Maybe it's helpful for someone. Most probably that will be myself when I'm not near the local copy.

Hall of Syntax:

1. <a href="#1">bash stuff</a>
 - <a href="#11">source and export</a>
 - <a href="#12">arguments</a>
 - <a href="#13">redirections and piping</a>
 - <a href="#14">conditional expressions</a>
 - <a href="#15">loops</a>
 - <a href="#16">using functions in bash</a>
 - <a href="#17">some quite useful command syntax</a>
2. <a href="#2">miscellanous</a>
 - <a href="#21">Matlab: Manually plot connected dots</a>
 - <a href="#22">OS X: bash script toggling visibility of hidden files in Finder</a>
 - <a href="#23">OS X: bash script toggling the OS X dashboard</a>
 - <a href="#24">editing DNS Records</a>
 - <a href="#25">Raspberry Pi Camera code snippets</a>
  - <a href="#251">take photos</a>
  - <a href="#252">rename photos</a>
  - <a href="#253">create video from photos</a>
  - <a href="#254">concat video files</a>
  - <a href="#255">convert to h264</a>
  - <a href="#256">cut video file</a>
3. <a href="#3">syntax mess</a>
4. <a href="#4">git stuff</a>
5. <a href="#5">synology stuff</a>
6. <a href="#6">vim stuff</a>

<a name="1"></a>

# bash stuff

<a name="11"></a>

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

<a name="12"></a>

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

<a name="13"></a>

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

<a name="14"></a>

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

<a name="15"></a>

## loops

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
    
<a name="16"></a>

## using functions in bash:

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

<a name="17"></a>

## some quite useful command syntax

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
    
    # replace string in all files, recursively
    find . -type f -exec sed -i 's/foo/bar/g' {} +

<a name="2"></a>

# miscellanous

<a name="21"></a>

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

<a name="22"></a>

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

<a name="23"></a>

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

<a name="24"></a>

## editing DNS Records

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

<a name="25"></a>

## Raspberry Pi Camera code snippets

<a name="251"></a>

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

<a name="252"></a>

### rename photos
    for file in *.JPG ; do mv "${file}" "${file%.JPG}_suffix.JPG";  done
    i=1;
    for file in cam_*.JPG ; do mv -v "${file}" "$i.jpeg"; i=$[i+1]; done

<a name="253"></a>

### create video from photos
    ~/bin/ffmpeg -r 20 -i %d.jpeg /Users/jens/Desktop/vid.mp4

<a name="254"></a>

### concat video files
    ffmpeg -v info -f concat -i files.txt  -c copy output.mp4
    with files.txt containing:
        file '2017_0515_010105_001.MP4'
        file '2017_0515_011328_002.MP4'

<a name="255"></a>

### convert to h264
    ffmpeg -i input.mp4 -vcodec h264 output.mp4

<a name="256"></a>

### cut video file
    ffmpeg -i input_video.mp4 -ss 00:02:15 -to 00:05:30 -c copy      output_video.mp4    # cut before 2:15 and after 5:30
    ffmpeg -i input.mkv       -ss 00:00:00  -t 00:30:00 -codec copy  out.mkv             # cut after 30 minutes

<a name="3"></a>

# syntax mess

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

    for i in `ls *.jpg`; do convert -resize 50% -quality 90 $i conv_$i; done 

    # crop all files and then... do some more things
    for i in `ls *.png`; do convert -verbose -crop 1352x970+985+219 $i 123/conv_$i; done
    i=0; while [ $i -lt 10 ]; do echo "mv seite_$i"; i=$[i+1]; done
    for name in *\ *; do echo \"$name\"; done
    
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




    openssl aes-256-cbc -e -a -in PLAIN -out ENCRYPTED
    openssl aes-256-cbc -d -a -in ENCRYPTED -out PLAIN

    git tag -a v1.0
    git tag -a v0.9 558151a
    git log –oneline –decorate –graph



    alias english=’export LANG=en_US’
    
    duck -e overwrite --password mylittlepony --upload sftp://ponyguy@server.com/path aLocalDir
    
    # you may add --user foo --password bar
    wget \
     --recursive \
     --no-clobber \
     --page-requisites \
     --html-extension \
     --convert-links \
     --restrict-file-names=windows \
     --domains snej.de \
     --no-parent \
    http://snej.de/
    
    sed -i.bak 's/OLDSTRING/NEWSTRING/g' *.cpp
    
    echo "A;B:C,D.E" | awk -F'[,;.:]' '{print $3}'
    
    # adding --delete to rsync will remove files from micro128 which are not existing in Documents
    rsync -ravzh Documents /Volumes/micro128/

    pandoc --number-sections -t html | pandoc -f html -t markdown 

    NRF51 Programming: Target BOOT

    "The book promotes a clean, side-effect-free style of programming for the first eight chapters. Chapter 9 discusses i/o."
    

<a name="4"></a>

# git stuff

## undo
    git reset HEAD~                  # undo last commit
    git reset HEAD~N                 # undo last N commits
    git reset --soft HEAD~           # undo last commit and keep staged stuff
    git reset --hard HEAD~           # undo last commit and discard changes
    git gc --prune=now --aggressive  # force git to run garbage collector

## rebase your branch to latest master on origin
    git checkout master
    git pull
    git checkout -                   # goes back to your branch
    git rebase master
    git push --force

## squash
All squash examples below will squash the last 5 commits

### via merge
    git reset --hard HEAD~5          # git back 5 commits 
    git merge --squash HEAD@{1}      # squash from here to HEAD
    git commit                       # editor will come up with all commit messages

### via reset
    git reset --soft HEAD~5 && git commit   # editor will come with empty commit message

### via interactive rebase
    git rebase -i HEAD~5             # then select s (squash) for the commits to squash

## amend multiple commits
    git rebase -i HEAD~5             # then select e (edit) for the commits to amend
                                     # then call git commit --amend && git rebase --continue
                                     # ...until all commits have been amended

## Checkout master from origin
    git checkout -B master origin/master
    
## Pull without auto-merge
    git pull --rebase
    
## Stage hunks interactively
    git add -p <filename>
    
## delete remote tags & branches
    git push --delete origin v1.0       # deletes the a branch or tag named "v1.0"
    git push origin :refs/tags/v1.0     # explicitly delete a tag (no confusion with branches)

## change timestamp of last commit
    newdate="Tue 30 Mar 2021 12:34:56 GMT"; GIT_COMMITTER_DATE=$foosky git commit --amend --no-edit --date "$newdate"

<a name="5"></a>

# notes for my NAS (Synology DS218j)

## ssh stuff

It's not enough to copy your key to the ```authorized_keys``` file. Therefore:

    # avoid indexing of your data
    Control Pangel -> Indexing Service -> Media Indexing -> Indexed Folder -> remove all folders
    Note: Data in your ~/Drive folder will still be index... just don't use the ~/Drive folder :-)

    # enable ssh and rsync
    Control Panel -> Terminal & SNMP -> Terminal -> Enable SSH servie
    Control Panel -> File Services -> FTP -> Enable SFTP servie
    Control Panel -> File Services -> rsync -> Enable rsync service
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    chmod 755 /volume1/homes/$USERNAME

    # add this line in /etc/sshd/sshd_config:
    RSAAuthentication yes

    # uncomment this line in /etc/sshd/sshd_config:
    PubkeyAuthentication yes

    # restart
    sudo synoservicectl --reload sshd


<a name="6"></a>

# vim stuff

    # delete until a certain line
    :d167G         delete until line 167
	
    # search (%=all lines; g=all occurenves in each line)
    :%s/foo/bar/g  
	
    # column mode
    CTRL+SHIFT+V   y,d,…   SHIFT+i to insert text


