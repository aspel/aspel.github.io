---
layout: post
title: "How to send log events into Telegram"
date: 2018-08-31 16:04:17 +0000
categories: blog
location: Antarctica, Troll
description: This is simple manual about how to send log events into Telegram
---
---
This is simple manual how to send log events into **Telegram**

* [Telegram](https://telegram.org/) - Telegram messages are heavily encrypted and can self-destruct. Cloud-Based. Telegram lets you access your messages from multiple devices.

## Register Telegram bot
First of all you need to register your own tgbot https://core.telegram.org/bots

**tgbot_id**: 123123:AAF1lJ123mDfr

## Find your chat_id

* Sing in into your Tg app
* Find you own tgbot
* Send test message to your tgbot

After you should find your chat_id
```sh
curl "https://api.telegram.org/bot123123:AAF1lJ123mDfr/getUpdates"|jq .

{
  "ok": true,
  "result": [
    {
      "update_id": 12345,
      "message": {
        "message_id": 21,
        "from": {
          "id": 333333,
          "is_bot": false,
          "first_name": "aaaaa",
          "language_code": "en-us"
        },
        "chat": {
          "id": 333333,
          "first_name": "aaaaa",
          "type": "private"
        },
        "date": 1535731935,
        "text": "test message"
      }
    }
  ]
}
```
Your **chat_id** is 333333

## Receive messages

Create script **echo_tg_bot.sh**

```sh
#!/bin/bash

CHATID="333333"
KEY="123123:AAF1lJ123mDfr"
TIME="10"
URL="https://api.telegram.org/bot$KEY/sendMessage"
TEXT=$(tee)

curl -s --max-time $TIME -d "chat_id=$CHATID&disable_web_page_preview=1&text=$TEXT" $URL >/dev/null
```

Test this script

```sh
echo 123|/home/ubuntu/echo_tg_bot.sh
```

Additional examples

```sh
/home/ubuntu>crontab -l

# m h  dom mon dow   command

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

0   2   *   *   *   find /st -mtime +14 -exec rm {} \; |/home/ubuntu/echo_tg_bot.sh
```
