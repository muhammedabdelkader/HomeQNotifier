# HomeQNotifier

short & smart HomeQ notifier bash script. I am looking for firsthand apartment to my family on `https://homeq.se/`, and decided to have it somehow automated notify me by new listing based on my search criteria. 

## Store HomeQNotifier.sh 

```bash
#! /bin/bash

### Fill the sendTo with all recipients comma separated
sendTo="rec1@gmail.com,rec12@gmail.com"

### Change Search Params as fitting your pref 
query='{"min_room":"3","min_rent":"9000","min_area":"70","shapes":["metropolitan_area.7"],"sorting":"publish_date.desc","short_lease":false,"tags":["everyone"]}'

## File names 
originalFile="./currentHomeQ"
tempFile="/tmp/test"
[ -f $originalFile ] && echo "[!] $originalFile exist." || echo > $originalFile

## Mail subject set to DateTime each time 
subjectR=$(date | sed 's/ //g')

## Execution 
curl 'https://search.homeq.se/api/v2/search'   -H 'Connection: keep-alive'   -H 'sec-ch-ua: "Google Chrome";v="95", "Chromium";v="95", ";Not A Brand";v="99"'   -H 'Accept: application/json, text/plain, */*'   -H 'Content-Type: application/json;charset=UTF-8' -H 'sec-ch-ua-mobile: ?0'   -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36'   -H 'sec-ch-ua-platform: "macOS"'   -H 'Origin: https://www.homeq.se'   -H 'Sec-Fetch-Site: same-site'   -H 'Sec-Fetch-Mode: cors'   -H 'Sec-Fetch-Dest: empty'   -H 'Referer: https://www.homeq.se/'   -H 'Accept-Language: en-GB,en-US;q=0.9,en;q=0.8,sv;q=0.7'   --data-raw $query  --compressed | jq ".results[]|select (.rent < 20000)| select (.amenities.dishwasher==true and .amenities.washer==true and .amenities.drier==true and .amenities.elevator==true)" | jq '{"rent":.rent,"date":.publish_date,"amenities":.amenities,"url":("https://www.homeq.se/lagenhet/"+(.id|tostring)+"-"+(.rooms|tostring)+"rum-"+(.address.municipality)+"-stockholms-lan-"+(.address.street)), "rooms":.rooms, "add":.address, "Display":.diplay,"id":.id}' | base64  > $tempFile

# Stop redundancy 
if [ $(diff $tempFile $originalFile | wc -l) -ne 0 ];then
        cat $tempFile > $originalFile
        cat $tempFile | base64 -d | mail -aFrom:no-reply@digitalOcean.redactive.io -r noreply@digitalocean.redactive.io -s 'HomeQ:'$subjectR $sendTo
fi
```

## Install smtp server 

```bash
sudo apt-get install -y mailutils
```

## Configure `/etc/ssmtp/ssmtp.conf`

```
root=postmaster
#mailhub=mail
FromLineOverride=YES
UseSTARTTLS=YES
FromLineOverride=YES
root=<anyemail>
mailhub=smtp.gmail.com:587
AuthUser=<email used for sending out emails>
AuthPass=<email password>
```

## Configure Crontab 

1. Edit crontab `crontab -e`

2. Add record `0 * * * 1-5 <full Path to bash script > >/dev/null 2>&1` _run every hour during working days_

3. add execution permission `chmod a+x <full path to the script>`
