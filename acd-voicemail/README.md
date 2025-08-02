# ACD Voicemail Player

Currently the ability to play the voice mail that is routed as part of an ACD interaction is road mapped so in the meantime I have created a simple HTML page that can either be hosted externally eg GCP, AWS or Azure or to make like easier you can also simply do this as a `Static Resource` in Salesforce and render it directly from there in a `Visual Force Page`.

![](images/player.png?raw=true)

This repo assumes you have experience with both Genesys Cloud admin as well as Salesforce.

## OAuth setup

The OAuth for this HTML pages works the same as most of my other client side examples in my GitHub as its based on the [user snippets](https://github.com/mcphee11/vscode-user-snippets) you need to setup a PKCE Auth in Genesys Cloud with the required permissions and redirect URL. The redirect URL will differ based on where your hosting the file.

![](images/auth.png?raw=true)

Once you have created the auth copy the

```
clientId
```

## Hosting in Salesforce

In this example I think it makes the most sense to host it as a `Static Resource` inside salesforce. To do this go to the settings and upload the HTML file there.

![](images/hosting.png?raw=true)

```
NOTE: the name you use will be referenced later
```

## ConversationId

For this integration I have kept the code VERY simple and am getting the `conversationId` from the voice object. For this it needs to be mapped. I have a custom field on my voice object called `gc_conversationId__c` if you do create a new field for this ensure it supports enough string chars for the entire UUID.

you then need to 'map' the conversationId to the new field, there is a OOTB item for conversationId with CX Cloud. Go into the CX Cloud admin and setup the mapping:

![](images/mapping.png?raw=true)

details on the mapping options can be found [here](https://help.mypurecloud.com/articles/synchronize-call-attributes-with-salesforce-activity-records-for-cx-cloud-from-genesys-and-salesforce/)

## Visualforce Page

Now you need to create teh visual force page itself. I have put my example below but you will need to replace the parts where your own domain is required as well as if you have a differently named field you are mapping the `conversationId` to.

```
<apex:page standardController="VoiceCall">
    <iframe id="voiceMailPlayer" width="100%" frameborder="0" height="100px" src="{!$Resource.voiceMailPlayer}?gc_region=mypurecloud.com.au&gc_clientId=ENTER_YOUR_CLIENT_ID&gc_conversationId={!VoiceCall.gc_conversationid__c}&gc_redirectUrl=https://ENTER_YOUR_DOMAIN.force.com{!$Resource.voiceMailPlayer}">
        <p>Genesys Voicemail Player cannot be displayed. Your browser does not support iframes.</p>
    </iframe>
</apex:page>
```

Ensure that you have updated both the

```
ENTER_YOUR_CLIENT_ID
ENTER_YOUR_DOMAIN
```

As well as setup the entire redirect URL in the Genesys Cloud Auth where the clientId was created.

## Show the player

Now all you need to do is put the visual force page in a component of the voice call object that makes sense. Personally I put it in the same place as the `callback` LWC as when a ACD voice mail is routed it comes through as a callback object.

## NOTES

Depending on the hosting solution you may need to allow CORS. In salesforce this is done inside `Trusted Sites` if your hosting on GCP etc then they have their own setup for this etc.
