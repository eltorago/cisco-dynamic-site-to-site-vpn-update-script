#!/bin/bash
#### This script contains mostly commands I have stolen from Stack Exchange and a few other websites
#### Thanks to ptrimboli for the help on checking the novpn part
#### 2015 - tgarbin	

#### To Do: make a log file of the process for debugging

### Run this script as the rancid user or the user with rancid access.

SCRIPT_HOME=/home/rancid/scripts

DEBUGMODE=N
LOG_FILE=$SCRIPT_HOME/vpn/vpn.log

if [ "$DEBUGMODE" = Y ]; then
 echo -e "\e[31mDEBUG\e[0m: $0: about to parse datafile;" 1>&2
 echo -e "Filename is [\e[35m$LOG_FILE\e[0m]" 1>&2
 trap "echo $0: [\$LINENO]" DEBUG
fi

touch $LOG_FILE
exec 3>&1 1>>${LOG_FILE} 2>&1

## Go to Working Directory
cd /home/rancid

## Modify these variables to your specific needs:
## Rancid specific variables
## Rancid clogin
RANCID_CLOGIN=/home/rancid/bin/clogin
## Rancid run
RANCID_RUN=/home/rancid/bin/rancid-run
## Rancid Groups
GROUP1=''
GROUP2=''
#etc...
## Crypto map
CRYPTO_MAP=''
##Crypto isakmp peer 0 key - please tell me how you can use 6 peer key, it doesn't seem to work.
VPN_KEY=''
## Your router Hostname/Address
LOCAL_ROUTER=''

## Your peer's internal Hostname/Address
PEER_ROUTER_INTERNAL_HOSTNAME=''
## Your peer's external Hostname/Address
PEER_ROUTER_EXTERNAL_HOSTNAME=''

## Run rancid to capture differences
PROSTERITY () {
`$RANCID_RUN $GROUP1 $GROUP2`
}

## General variables
INTERNAL_PING=`ping -c 5 $PEER_ROUTER_INTERNAL_HOSTNAME | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | awk -F'[ :]' 'NR==5'`
PEER_ROUTER_EXTERNAL_IP=`ping -c 5 $PEER_ROUTER_EXTERNAL_HOSTNAME | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | awk -F'[ :]' 'NR==5'`
VPN_CONFIG_DELETED_FILE=$SCRIPT_HOME/vpn/VPN_CONFIG_DELETED_FILE
VPN_DOWN_FILE=$SCRIPT_HOME/vpn/VPN_DOWN_FILE
CONFIG_DEJA_VU=$SCRIPT_HOME/vpn/CONFIG_DEJA_VU
PEER_LAST_KNOWN_IP_LOG=$SCRIPT_HOME/vpn/PEER_LAST_KNOWN_IP_LOG
PEER_LAST_KNOWN_IP=$(cat $PEER_LAST_KNOWN_IP_LOG)
OLD_PEER_ROUTER_EXTERNAL_IP_LOG=$SCRIPT_HOME/vpn/OLD_PEER_ROUTER_EXTERNAL_IP_LOG.txt
OLD_PEER_ROUTER_EXTERNAL_IP=$(cat $OLD_PEER_ROUTER_EXTERNAL_IP_LOG)
WHERE_IS_DATE=`which date`
date=$($WHERE_IS_DATE)

# Mail variables
MAIL_SERVER_IP=127.0.0.1
MAIL_SERVER_PORT=25
MAIL_RECIPIENT=''
MAIL_SENDER=''

## Functions
### Mail Functions
PEER_UPDATE_SUCESS_TEMPLATE () {
SUBJECT="Branch IP has changed."
BODY="The Branch router has aquired a new IP address on $date
New IP: $NEW_PEER_ROUTER_EXTERNAL_IP
Old IP: $OLD_PEER_ROUTER_EXTERNAL_IP
Last Known WAN IP: $PEER_LAST_KNOWN_IP"
}

PEER_ROUTER_IS_DOWN_TEMPLATE () {
SUBJECT="Branch Router is down!"
BODY="The branch router cannot be reached on $date"
}

VPN_REMOVAL_FAILURE_TEMPLATE () {
SUBJECT="Cant remove VPN config!"
BODY="The local router cannot be reached on $date"
}

VPN_UPDATE_FAILURE_TEMPLATE () {
SUBJECT="Cant update VPN config!"
BODY="The local router cannot be configured on $date"
}

PEER_ROUTER_BACK_UP_TEMPLATE () {
SUBJECT="VPN back up!"
BODY="The VPN has been re-established on $date"
}

## Email template
BASE_EMAIL_TEMPLATE () {
local recipient=$MAIL_RECIPIENT
nc $MAIL_SERVER_IP $MAIL_SERVER_PORT << EOF
ehlo mail.script
mail from:<$MAIL_SENDER>
rcpt to:<$MAIL_RECIPIENT>
data
subject: $SUBJECT
$BODY

VPN Script
.
quit
EOF
}


SENDING_PEER_UPDATE_SUCCESS_EMAIL () { PEER_UPDATE_SUCESS_TEMPLATE; BASE_EMAIL_TEMPLATE; }

SENDING_PEER_ROUTER_DOWN_EMAIL () { PEER_ROUTER_IS_DOWN_TEMPLATE; BASE_EMAIL_TEMPLATE; }

SENDING_VPN_REMOVAL_FAILURE_EMAIL () { VPN_REMOVAL_FAILURE_TEMPLATE; BASE_EMAIL_TEMPLATE; }

SENDING_VPN_UPDATE_FAILURE_EMAIL () { VPN_UPDATE_FAILURE_TEMPLATE; BASE_EMAIL_TEMPLATE; }

SENGING_PEER_ROUTER_BACK_UP_EMAIL () { PEER_ROUTER_BACK_UP_TEMPLATE; BASE_EMAIL_TEMPLATE; }

## Clogin Functions

RANCID_REMOVE_CONFIG () {
if [[ $( ($RANCID_CLOGIN -c "conf t; no crypto isakmp key 0 $VPN_KEY address $OLD_PEER_ROUTER_EXTERNAL_IP no-xauth; crypto map $CRYPTO_MAP 1 ipsec-isakmp; no set peer $OLD_PEER_ROUTER_EXTERNAL_IP; end; wr;" $LOCAL_ROUTER) | grep 'Error:') ]]; then

{
echo "...And I can't even access the router"
echo "Sending email of failure"
} 1>&2
 
## Email config removal failure.
SENDING_VPN_REMOVAL_FAILURE_EMAIL; exit

else

## Make a file to be checked next time the lookup fails. If found, the remove config step will be skipped and it will just end script.
 touch $VPN_CONFIG_DELETED_FILE
## Remove the IP in the log so it stops updating if it comes back up with the same IP
 echo "" > $OLD_PEER_ROUTER_EXTERNAL_IP_LOG
 echo "Removed old config, ending script" 
# Run rancid
`$RANCIDRUN $GROUP1 GROUP2`
## Email config removal and router down notice.
SENDING_PEER_ROUTER_DOWN_EMAIL; exit

fi
}

RANCID_REMOVE_DEJA_VU () {
if [[ $( ($RANCID_CLOGIN -c "conf t; no crypto isakmp key 0 $VPN_KEY address $OLD_PEER_ROUTER_EXTERNAL_IP no-xauth; crypto map $CRYPTO_MAP 1 ipsec-isakmp; no set peer $OLD_PEER_ROUTER_EXTERNAL_IP; end; wr;" $LOCAL_ROUTER) | grep 'Error:') ]]; then

{
echo "...And I can't even access the router"
echo "Sending email of failure"
} 1>&2

## Email config removal failure.
SENDING_VPN_REMOVAL_FAILURE_EMAIL; exit

else

## Make a file to be checked next time the lookup fails. If found, the remove config step will be skipped and it will just end script.
 touch $VPN_CONFIG_DELETED_FILE;
## Remove the IP in the log so it stops updating if it comes back up with the same IP
 echo "" > $OLD_PEER_ROUTER_EXTERNAL_IP_LOG;

if [ "$PEER_ROUTER_EXTERNAL_IP" != "$(cat $CONFIG_DEJA_VU)" ]; then
## Email config removal and router down notice.
 echo "" > $CONFIG_DEJA_VU;
 echo "Removed old config, ending script and emailing." 1>&2;

# Run rancid
PROSTERITY

SENDING_PEER_ROUTER_DOWN_EMAIL; exit

else

echo "The WAN IP $PEER_ROUTER_EXTERNAL_IP is the same as when email was sent and the tunnel is still down, ending script and preventing spam emails about it" 1>&2

 fi

fi
}

RANCID_ADD_NEW_CONFIG () {
if [[ $( ($RANCID_CLOGIN -c "conf t; crypto isakmp key 0 $VPN_KEY address $PEER_ROUTER_EXTERNAL_IP no-xauth; crypto map $CRYPTO_MAP 1 ipsec-isakmp; set peer $PEER_ROUTER_EXTERNAL_IP; end; wr;" $LOCAL_ROUTER) | grep 'Error:') ]]; then

# To Do: use push notification )Y)
  echo "Router Update Failed! Sending email..." 1>&2

## Email config update failure.
SENDING_VPN_UPDATE_FAILURE_EMAIL; exit

else

rm $VPN_CONFIG_DELETED_FILE

# Run rancid
PROSTERITY

fi
}

RANCID_UPDATE_CONFIG () {
if [[ $( ($RANCID_CLOGIN -c "conf t; no crypto isakmp key 0 $VPN_KEY address $OLD_PEER_ROUTER_EXTERNAL_IP no-xauth; crypto isakmp key 0 $VPN_KEY address $PEER_ROUTER_EXTERNAL_IP no-xauth; crypto map $CRYPTO_MAP 1 ipsec-isakmp; no set peer $OLD_PEER_ROUTER_EXTERNAL_IP; set peer $PEER_ROUTER_EXTERNAL_IP; end; wr;" $LOCAL_ROUTER) | grep 'Error:') ]]; then

# To Do: use push notification )Y)
  echo "Router Update Failed! Sending email..." 1>&2

## Email config update failure
SENDING_VPN_REMOVAL_FAILURE_EMAIL; exit

elif [ -e "$VPN_CONFIG_DELETED_FILE" ]; then

rm $VPN_CONFIG_DELETED_FILE

# Run rancid
PROSTERITY

fi
}



########################
###### MAIN SCRIPT #####
########################

## Check if IP address resolve has failed and returned null value. If so login and remove the old config.
if [ -z "$PEER_ROUTER_EXTERNAL_IP" ]; then

 echo "Oh Dear! I could not resolve the IP." 1>&2

if [ -e "$VPN_CONFIG_DELETED_FILE" ]; then

 echo "No config removal needed, ending script."; 1>&2

exit

else

RANCID_REMOVE_CONFIG

 fi

fi

## Check if IP address has changed
if [ "$PEER_ROUTER_EXTERNAL_IP" != "$OLD_PEER_ROUTER_EXTERNAL_IP" ]; then

echo "IP change! Configuring router for VPN update" 1>&2

## Check if IP address resolve has failed and returned null value. If so the tunnel is not up so remove the config.
elif [[ "$PEER_ROUTER_EXTERNAL_IP" == "$OLD_PEER_ROUTER_EXTERNAL_IP" && ! -z "$INTERNAL_PING" && -e "$VPN_DOWN_FILE" ]]; then

echo "VPN back up, sending email..." && SENGING_PEER_ROUTER_BACK_UP_EMAIL

rm $VPN_DOWN_FILE; exit

elif [[ "$PEER_ROUTER_EXTERNAL_IP" == "$OLD_PEER_ROUTER_EXTERNAL_IP" && ! -z "$INTERNAL_PING" ]]; then

echo "No IP change, no config change required, Tunnel still up!" 1>&2; exit

else


# I think the DEJAVU variable needs t be here to cehck and invoke that function to stop the sapm...

echo "Oh Dear! The IP hasn't changed but I could not resolve internal branch IP. Something is wrong." 1>&2

## Need to make a file somewhere to check next time it happens. so if it fails and emails me then come back up on its own, let me know. 
touch $VPN_DOWN_FILE

SENDING_PEER_ROUTER_DOWN_EMAIL; exit

fi

## Check to see if the outdated config doesn't exist. if so, run RANCID_ADD_NEW_CONFIG whic just adds the new config.
if [ -e "$VPN_CONFIG_DELETED_FILE" ]; then

RANCID_ADD_NEW_CONFIG

## Check to see if the outdated config does exist. if so, run RANCID_UPDATE_CONFIG which removed ths old config and then adds the new config.
elif [ ! -f "$VPN_CONFIG_DELETED_FILE" ]; then

RANCID_UPDATE_CONFIG

fi

## If either of the previous two commands execute successfully, do this.
echo "Update Success! Pinging internal IP to see if VPN is up..." 1>&2
echo "Sleeping for 5" 1>&2
sleep 5

## Contextually relevent log variables.
## Save last failed IP so i dont get email spam if the external ip does not change.
echo "$PEER_ROUTER_EXTERNAL_IP" > $PEER_LAST_KNOWN_IP_LOG
## We "assume" for now that we have the correct new IP, so change the ip log...
#echo "$PEER_ROUTER_EXTERNAL_IP" > $OLD_PEER_ROUTER_EXTERNAL_IP_LOG

## Contextual based variable
INTERNAL_PING_VERIFY=`ping -c 5 $PEER_ROUTER_INTERNAL_HOSTNAME | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | awk -F'[ :]' 'NR==5'`

## Check if the internal branch IP address ping succeeds. If not so that is not my IP, the tunnel is not up so remove the config.
if [ -z "$INTERNAL_PING_VERIFY" ]; then

 echo "Oh Dear! I could not resolve internal branch IP, [$PEER_ROUTER_EXTERNAL_IP] must not be our real WAN IP." 1>&2

RANCID_REMOVE_DEJA_VU

## Save last failed IP so i dont get email spam if the external ip does not change.
echo "$PEER_ROUTER_EXTERNAL_IP" > $CONFIG_DEJA_VU; exit

## Check if the internal branch IP address ping succeeds. If so, echo the success.
elif [ ! -z "$INTERNAL_PING_VERIFY" ]; then

echo "VPN is UP/UP" 1>&2
echo "Sending New IP [$PEER_ROUTER_EXTERNAL_IP] and VPN confrimation email..." 1>&2

  if [ -e "$VPN_DOWN_FILE" ]; then

    rm $VPN_DOWN_FILE

  fi

echo "$PEER_ROUTER_EXTERNAL_IP" > $OLD_PEER_ROUTER_EXTERNAL_IP_LOG

## Email specific contextually important variable
NEW_PEER_ROUTER_EXTERNAL_IP=$(cat $OLD_PEER_ROUTER_EXTERNAL_IP_LOG)
## Email IP and VPN Success
SENDING_PEER_UPDATE_SUCCESS_EMAIL

fi
