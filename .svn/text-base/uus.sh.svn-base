#!/bin/bash

########################################################
# uus.sh                                               #
# Unmaned Update Script (UUS)                          #
#                                                      #
# Usage: sh ./uus.sh <oldimage> <newimage> [ -v [ # ]] #
#                                                      #
# Bash header script to manage the run of the parsing  #
# awk script.  This just is a clean way to run and    #
# make sure everything is set up correctly             #
#                                                      #
# @author David Walker (azrail@csh.rit.edu)            #
#                                                      #
# Version 1.0                                          #
########################################################

OLDMNT=/tmp/oldmnt
NEWMNT=/tmp/newmnt

# Check that there are at least 2 argyments
if [ $# -lt 2 ]; then
    echo "Usage: ./$0 <base image> <new image> [ -v [ # ]]"
    exit
fi

# Make sure that the base image exists and is a file.
if [ ! -e $1 ] || [ ! -f $1 ]; then
    echo "Base image file does not exist"
    exit
fi

# Make sure that the new image is both exists and is a file.
if [ ! -e $2 ] || [ ! -f $2 ]; then
    echo "New image file does not exist"
    exit
fi

if [ ! -e "../sha1_hash/sha1_hash" ]; then
	make -C "../sha1_hash"
	if [ $? != 0 ]; then
		exit
	fi
fi

# Verbose setting, default to 0
verb=0

# See if the third argument exists.  If it does, we will see if it is -v
# if it is then we set verbose mode to 1
if [ $3 ]; then
    if [ $3 == "-v" ]; then
        verb=1

        # If the user entered a number after -v
        if [ $4 ]; then
            # Set verbose level to what the user entered
            lvl=$4
        else
            # Default verbose level
            lvl=1
        fi
    fi
fi

# See if the target mount directory exists
if [ -e $OLDMNT ]; then
    # It does, now make sure it is not mounted.
    umount $OLDMNT 2> /dev/null
else
    # It does not, make it.
    mkdir $OLDMNT 2> /dev/null
fi

# See if the target mount directory exists
if [ -e $NEWMNT ]; then
    # It does, now make sure it is not mounted.
    umount $NEWMNT 2> /dev/null
else
    # It does not, make it.
    mkdir $NEWMNT 2> /dev/null
fi

# See if the update folder exists.  This will be deleted if it does
# because the AWK script will create and populate it.
if [ -e "./bundle" ]; then
    rm -fR ./bundle
fi

# Mount the 2 images
mount -o loop $1 $OLDMNT
mount -o loop $2 $NEWMNT

# If we have verbose set
if [ $verb == 1 ]; then
    # Run the AWK script
    ls -AliR $OLDMNT $NEWMNT | awk -f uus.awk -v oldmnt=$OLDMNT \
       -v newmnt=$NEWMNT -v dbg=$lvl
else # No verbose mode
    # Run the AWK script
    ls -AliR $OLDMNT $NEWMNT | awk -f uus.awk -v oldmnt=$OLDMNT \
       -v newmnt=$NEWMNT
fi

# Generate Name.
#oldvers=`$OLDMNT/usr/bin/hs92XX --version | awk '{ print $2 }'`
newvers=`$NEWMNT/usr/bin/hs92XX --version | awk '{ print $2 }'`
#name=$oldvers"_to_"$newvers
name="to_"$newvers

# Clean up, unmount images
umount $OLDMNT 2> /dev/null
umount $NEWMNT 2> /dev/null

# Move into the update folder
cd bundle 

# Tar and bzip2 everything
tar -cjf $name.tar.bz2 *

# Make the SHA1hash of the bzip2 image
/usr/src/spectracom/tools/sha1_hash/sha1_hash $name.tar.bz2 > $name.tar.bz2.sha1

cd ..
