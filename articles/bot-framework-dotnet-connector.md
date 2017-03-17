---
title: Send and receive activities using the Bot Framework Connector and .NET | Microsoft Docs
description: Learn how to send and receive activities using the Bot Framework Connector via the Bot Builder SDK for .NET.
keywords: Bot Framework, .NET, Bot Builder, SDK, Connector, Connector service, activity, send activity, receive activity
author: kbrandl
manager: rstand
ms.topic: develop-dotnet-article
ms.prod: botframework
ms.service: Bot Builder
ms.date: 03/09/2017
ms.reviewer:
#ROBOTS: Index
---

# Send and receive activities using the Connector

The Bot Framework Connector provides a single REST API that enables a bot to communicate across multiple 
channels such as Skype, Email, Slack, and more. 
It facilitates communication between bot and user, by relaying messages from bot to channel 
and from channel to bot. 

This article describes how to use the Connector via the Bot Builder SDK for .NET to 
exchange information between bot and user on a channel. 

> [!NOTE]
> While it is possible to construct a bot by exclusively using the techniques that are described
> in this article, the Bot Builder SDK provides additional features like 
> [Dialogs](bot-framework-dotnet-dialogs.md) and [FormFlow](bot-framework-dotnet-formflow.md) that 
> can streamline the process of managing conversation flow and 
> make it simpler to incorporate cognitive services such as language understanding.

##<a id="create-client"></a> Create a connector client

The **ConnectorClient** class contains the methods that a bot uses to communicate with a user on a channel. 
When your bot receives an [Activity](bot-framework-dotnet-activities.md) object from the Connector, 
it should use the **ServiceUrl** specified for that activity to create the connector client that it'll 
subsequently use to generate a response. 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> Because a channel's endpoint may not be stable, your bot should direct communications to the endpoint 
> that the Connector specifies in the **Activity** object, whenever possible (rather than relying upon a cached endpoint). 
>
> If your bot needs to initiate the conversation, it can use a cached endpoint for the specified channel 
> (since there will be no incoming **Activity** object in that scenario), but it should refresh cached endpoints often. 

##<a id="create-reply"></a> Create a reply

The Connector uses an [Activity](bot-framework-dotnet-activities.md) object to pass information back and forth between bot and channel (user). 
Every activity contains information used for routing the message to the appropriate destination 
along with information about who created the message (**From** property), 
the context of the message, and the recipient of the message (**Recipient** property).

When your bot receives an activity from the Connector, the incoming activity's **Recipient** property specifies 
the bot's identity in that conversation. 
Because some channels (for example, Slack) assign the bot a new identity when it's added to a conversation, 
the bot should always use the value of the incoming activity's **Recipient** property as the value of 
the **From** property in its response.

Although you can create and initialize the outgoing **Activity** object yourself from scratch, 
the Bot Builder SDK provides an easier way of creating a reply. 
By using the incoming activity's **CreateReply** method, 
you simply specify the message text for the response, and the outgoing activity is created 
with the **Recipient**, **From**, and **Conversation** property automatically populated.

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## Send a reply

Once you've created a reply, you can send it by calling the connector client's **ReplyToActivity** method. 
The Connector will deliver the reply using the appropriate channel semantics. 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> If your bot is replying to a user's message, always use the **ReplyToActivity** method.

## Send a (non-reply) message 

If your bot is part of a conversation, it can send a message that is not a direct reply to 
any message from the user by calling the **SendToConversation** method. 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

> [!TIP]
> You may use the **CreateReply** method to initialize the new message (which would automatically set 
> the **Recipient**, **From**, and **Conversation** properties for the message). 
> Alternatively, you could use the **CreateMessageActivity** method to create the new message 
> and set all property values yourself.

> [!NOTE]
> The Bot Framework does not impose any restrictions on the number of messages that a bot may send. 
> However, most channels enforce throttling limits to restrict bots from sending a large number of messages in a short period of time. 
> Additionally, if the bot sends multiple messages in quick succession, 
> the channel may not always render the messages in the proper sequence.

## Start a conversation

There may be times when your bot needs to initiate a conversation with one or more users. 
You can start a conversation by calling either the **CreateDirectConversation** method (for a private conversation with a single user) 
or the **CreateConversation** method (for a group conversation with multiple users) 
to retrieve a **ConversationAccount** object. 
Then, create the message and send it by calling the **SendToConversation** method.

> [!NOTE]
> To use either the **CreateDirectConversation** method or the **CreateConversation** method,
> you must first [create the connector client](#create-client) by using the target channel's service URL 
> (which you may retrieve from cache, if you've persisted it from previous messages). 

> [!NOTE]
> Not all channels support group conversations. 
> To determine whether a channel supports group conversations, consult the channel's documentation.

The following code example uses the **CreateDirectConversation** method to create a private conversation with a single user.

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

The following code example uses the **CreateConversation** method to create a group conversation with multiple users.

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## Additional resources

- [Create messages](bot-framework-dotnet-create-messages.md)
- [Add attachments to messages](bot-framework-dotnet-add-attachments.md)
- [Implement channel-specific functionality](bot-framework-dotnet-channeldata.md)