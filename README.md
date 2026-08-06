<p align="center">
	<img height="300" src="https://s.alicdn.com/@sc01/kf/H55421308235b45568c523f2684731916J.png">
</p>



# Proscenic M7 Pro ( Home Assistant package)

Since I purchased the Proscenic M7 PRO my goal has been to integrate it into Home Assistant.
Unfortunately, there are no solutions for a direct integration, as only the previous models (e.g. model 790t) have been integrated.
Speaking with some developers I was suggested to create a local proxy, with which to intercept calls made by the application for smartphones, so I can replicate these calls and finally integrate the vacuum in the home automation system.

After a lot of work I was able to intercept such information.

I have noticed an increasing interest in this vacuum model, so I decided to speed up a little bit and start publishing what I did. 

Almost all the functions have been integrated, something is still missing. I hope to be able to publish as many things as possible in the shortest time possible.

All your help is precious, feel free to collaborate. Thanks


## Installation

1. Download the folder `proscenic_m7_pro` and paste it into the `packages` folder.
Package configuration must be enabled. Just add to your `configuration.yaml`:
    ```yaml
    homeassistant:
      packages: !include_dir_named packages
    ```

2. Now open a linux terminal and run the following commands:
    ```bash
    read -p "Please enter your Email/Username: " LOGINUSER
    read -p "Please enter your Password: " PASSWORD

    curl -v -k -X POST -H "os: i" -H "Content-Type: application/json" -H "c: 338" -H "lan: en" -H "Host: mobile.proscenic.tw" -H "User-Agent: ProscenicHome/1.7.8 (iPhone; iOS 14.2.1; Scale/3.00)" -H "v: 1.7.8" -d "{\"state\":\"欧洲\",\"countryCode\":\"49\",\"appVer\":\"1.7.8\",\"type\":\"2\",\"os\":\"IOS\",\"password\":\"$(echo -n $PASSWORD | md5sum)\",\"registrationId\":\"13165ffa4eb156ac484\",\"language\":\"EN\",\"username\":\"$LOGINUSER\",\"pwd\":\"$PASSWORD\"}" "https://mobile.proscenic.tw/user/login"
    ```

    > Obviously it is necessary to enter your `LOGINUSER` and `PASSWORD` with the relative access data to the Proscenic Home application.

    You will get a response like this:
    ```
    {
      "code": 0,
      "msg": "success",
      "data": {
        "token": "XXXXXXXXXXXXXXX",
        "uid": "XXXXXXXXXXXXXXX",
        "equipcount": 0,
        "nickname": null,
        "recieveMsg": false,
        "countryCode": "49",
        "homeId": null
      }
    }
    ```

3. Then run
    ```bash
    curl "https://mobile.proscenic.com.de/user/getEquips/$LOGINUSER"  -d "username=$LOGINUSER"
    ```
    You will get a response like this:
    ```
    "content": [
          {
            "name": "M7 Pro",
            "code": "M7_PRO",
            "typeName": "CleanRobot",
            "model": "811_LDS",
            "sn": "XXXXXXXXXXXXXXX",
            "deviceId": null,
            "status": true,
            "imgUrl": "http://mobile.proscenic.com.de/images/M7_PRO.png",
            "homeId": null,
            "shared": false,
            "jump": "M7_PRO",
            "ctrlversion": null,
            "enabled": true,
            "scMac": null,
            "scSV": null,
            "faqUrl": "https://www.proscenic.com/support/faq-f0460-l9999.html",
            "cloud": 0,
            "type": "CleanRobot"
          }
    ]
    ```

4. Now insert your username, the token (from the first *curl* command) and the sn (from the second *curl* command) you just obtained into the `secrets.yaml` file contained within the `proscenic_m7_pro` folder.

---

## Available functions
When your restart Home Assistant you will be able to:
* through the vacuum entity:
  * start a normal cleaning process
  * pause the cleaning process (only pause, not continue. you need to start the cleaning process again)
  * send the vacuum to the charging base
  * set the fan speed to "quiet", "standard" or "powerful" 

* through the scripts:
  * start a deep cleaning
  * collect dust (additional dust bin needed)
  * continue the cleaning process
  * (obviously) all the functions available in the vacuum entity

## Lovelace manual card
Just create a new manual card and paste the following code
```
type: entities
entities:
  - entity: vacuum.proscenic_m7_pro
  - entity: script.proscenic_clean
  - entity: script.proscenic_deep_cleaning
  - entity: script.proscenic_pause
  - entity: script.proscenic_continue
  - entity: script.proscenic_charge
  - entity: script.proscenic_collect_dust
  - entity: script.proscenic_powermode_quiet
  - entity: script.proscenic_powermode_standard
  - entity: script.proscenic_powermode_powerful
show_header_toggle: false
title: Proscenic M7 PRO
```

