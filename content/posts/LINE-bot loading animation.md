---
title: "Adding the newest Loading Animation feature to LINE Bot"
date: 2024-08-21T10:55:32+08:00
tags:
 - LINE-messaging-API
 - LINE-bot
 - java
draft: false
---
### Background
I was integrating internal machine learning features to our official account bot, specifically STT and OCR services. The long pending time may cause confusion and uncertainty to the user. Adding a loading animation can help to improve the user experience.

### API Guide
We can add a loading animation via sending a request. Here is an example from API doc using `curl`:  

```bash
curl -v -X POST https://api.line.me/v2/bot/chat/loading/start \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer {channel access token}' \
-d '{
    "chatId": "U4af4980629...",
    "loadingSeconds": 5
}'
```
#### Channel Access Token
The `channel access token` can be found in the   

[LINE Developer Console](https://developers.line.biz/en/) > Console Home > Your Channel(needs to be Messaging API) > Messaging API  

and scroll to the bottom
![alt text](../../images/LINE-bot-channel_access_token.png)

#### ChatID
The `chatID` is in fact the userID. When someone sends a message to your LINE Official Account, LINE sends the webhook to your bot server. The ID is included in the webhook payload.

```json
{
    "destination": "U80xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "events": [
        {
            "type": "message",
            "message": {
                "type": "text",
                "id": "522...",
                "quoteToken": "88ZZMsqHVtJH......",
                "text": "你好\n"
            },
            "webhookEventId": "${webhookEventId}",
            "deliveryContext": {
                "isRedelivery": false
            },
            "timestamp": 1723009635825,
            "source": {
                "type": "user",
                "userId": "U31e......"
            },
            "replyToken": "${replyToken}",
            "mode": "active"
        }
    ]
}
```

#### Loading Seconds
The `loadingSeconds` is the duration of the loading animation. The maximum value is 60 seconds.
### Calling the API
We can call the API in Java using this example code:

```java
private OkHttpClient client = new OkHttpClient();

private void startLoadingAnimation(String userId, int seconds) {
    String payload = String.format("{\"chatId\":\"%s\",\"loadingSeconds\":%d}", userId, seconds);
    String channelAccessToken = ${channel access token};
    
    RequestBody body = RequestBody.create(payload, JSON);
    Request request = new Request.Builder()
        .url("https://api.line.me/v2/bot/chat/loading/start")
        .post(body)
        .addHeader("Authorization", "Bearer " + channelAccessToken)
        .build();
    
    try (Response response = client.newCall(request).execute()) {
        if (response.code() != 202) {
            log.error("Failed to start loading animation: {}", response.body().string());
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



### Reference
[LINE Messaging API official doc](https://developers.line.biz/en/docs/messaging-api/use-loading-indicator/)