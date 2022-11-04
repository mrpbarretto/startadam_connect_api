# startadam_connect_api
B2B API for StartADAM::Connect
StartADAM provides an API for creation of conversations between any application and Users in communication tools (comms-tools) such as Slack, Whatsapp, SMS, Telegram, MS Teams, Discord.
In what follows, a HostingApp is any application. For instance, a free-lancer marketplace which allows for hiring and management of execution of jobs. Or an ecommerce application. Or any other. 
# List of features

* [x] Add a User, which will be involved in Conversations later
* [x] Create a conversation between User
* [x] Get a copy of all messages and files exchanged between Users involved in Conversations  


# Installation

## Requirements
StartADAM integration API uses AWS SQS communication. You´ll need AWS provided SDKs for this integration. Check https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html.


# Documentation
## Part I: Introduction
### Concepts
Some concepts of startADAM must be kept in mind when reading this guide.

**1.Use of communication tool selected by end-user**
One key startADAM functionality is to allow the end user to choose her/his communication tool of preference. It´s on startADAM to echo from one to another, allowing for text, attachments, emojis, threaded conversations, etc. And to copy the HostingApp all communications, along with alerts such as sentiment analysis, improper conversations, etc.

The list of supported comms tools is being extended every day and already includes Slack, Discord, Telegram, Whatsapp, MS Teams, SMS. Most of these comms tools will require user authorization to allow communication. Therefore, user onboarding will normally require authorization to startADAM to join the conversation.

**2.Users model**
Different users model are supported by startADAM. For HostingApps, the “closed world” model is used. In this model, all users are registered by the HostingApp. All functionalities (for instance, sending messages) only happens via HostingApp. 

### Quick overview

StartADAM integration is composed by 3 main components:

* "**startADAM button**" is an iFrame to be embedded into a HostingApp´s webpage, to allow the user to selected the desired comms tool and complete the authorization process to allow startADAM to communicate with her/him. The intention is to easy onboarding, eliminating most of development efforts for this integration which has its complexities
* **async API** calls are the regular integration calls to invoke startADAM´s functionalities. Note it´s an async API (not a REST API) to allow to fully decouple the Marketplace from startADAM. These calls gives access to:
	* manage users (users can be added without the use of startADAM iFrame)
	* manage communication channel and message exchange

* **notifications** are messages send back to HostingApp, to notify events such as messages exchanged using comms-tools. 

All data transfer is encrypted end-to-end.
To get access to the queues, the startADAM DEVOPS team will work with your team to provide access-key and secret-key to configure.

## Part II: startADAM button

StartADAM iframe is an area reserved by the HostingApp with 300x300 pix (min) to be an iFrame where the User will be able to select a comms-tool and make the association between the HostingApp internal identification and the startADAM access to the comms-tool token for this user.

![](blob:https://fdte-adam.atlassian.net/64842c14-6800-463d-ac38-880bfccee12f#media-blob-url=true&id=0fb5f413-033b-4dea-9d62-2c9b67412232&collection=contentId-24903783&contextId=24903783&height=144&width=195&alt=)

StartADAM is responsible for all dialog between the User and the comms-tool to allow StartADAM to communicate with her/him.

## Part III: Async API : general aspects
StartADAM API is async, based on messages sent/received using AWS SQS. AWS SQS provides SDKs for various languages, easying the implementation. Each startADAM´s Client has its own set of queues (REQ for requests, RSP for responses, NOT for notifications) which are accessible via access credentials provided by startADAM support team.

### Response attributes

All responses have the following attributes.

| Property | Parent | Mandatory | Data type | Description |
| -------- | -------- | -------- | -------- | -------- |
| requestId | -- | yes |uuid | ID of request being processed, unique within Marketplace |
| source | -- |yes |string| Marketplace Key, assigned by startADAM |
| method | -- | yes | string | function name |
| returnCode |-- | yes | numeric | Error code, according to the following general list |
| returnMessage | -- | yes| string | Error message |

## Part IV: StartADAM::Connect

StartADAM::Connect includes calls to:

* create a communication channel between user and HostingApp
* send messages within a communication context, named as a channel

### Create conversation

This call creates a permanent communication channel between a user and the HostingApp. A channel will be provided according to the current communication tool in use by the user. If the user changes the communication tool, the channel changes accordingly. Communications occurs with distinct limitations through each communication tool; check the support for each communication tool on the specific startADAM documentation.


| Property | Parent | Mandatory | Data type | Description |
| -------- | -------- | -------- | -------- | -------- |
| accountId | -- | yes | string | Account ID for HostingApp. |
| channelName | -- | yes | string | Name of the channel that will be created |
| userReference | yes | string | User ID of the member according to the internal marketplace coding system |
| requestId | -- | yes | uuid | ID of request being processed, unique within HostingApp |
| source | -- | yes | string | HostingApp Key, assigned by startADAM. |
| method | -- | yes | string | Must be 'createChannel' |

### Send message or file

Send a text message with rich formatting or a file (photo, document, music, etc) to a user, using a previously created channel.

| Property | Parent | Mandatory | Data type | Description |
| -------- | -------- | -------- | -------- | -------- |
| accountId | -- | yes | string | Account ID |
| channelName| -- | yes | string | Name of the channel to be used |
| message | -- | no | string |Message to be sent |
| userReference | -- | yes | string | User ID of the member according to the internal marketplace coding system |
| attachments | -- | no | object array | Array of attachments |
| userReference | attachments | yes | string | User ID who created the Attachment, according to marketplace's internal system |
| externalSystem | attachments | yes | enum (url, binary) | Just file link or binary data |
| externalReference | attachments | yes | string | if externalSystem equals to URL, it must be an URL. Otherwise, it will be treated as filename |
| binaryData | attachments | no | string | mandatory if externalSystem equals to binary. It should contain base64 encoded content of file to be sent |
| mimeType | attachments | yes | string | MIME type for attachment |
| requestId | -- | yes | uuid | ID of request being processed, unique within HostingApp |
| source | -- | yes | string | HostingApp Key, assigned by startADAM. |
| method | -- | yes | string | Must be 'sendMessage' |

### Message exchange notification

Every time a message is exchange between users on a channel, the Marketplace is notified. It can contain a message or an attachment.

| Property | Parent | Mandatory | Data type | Description |
| -------- | -------- | -------- | -------- | -------- |
| accountId | -- | yes | string | Account ID |
| channelName| -- | yes | string | Name of the channel to be used |
| message | -- | no | string |Message to be sent |
| userReference | -- | yes | string | User ID of the member according to the internal marketplace coding system |
| attachments | -- | no | object array | Array of attachments |
| userReference | attachments | yes | string | User ID who created the Attachment, according to marketplace's internal system |
| externalSystem | attachments | yes | enum (url, binary) | Just file link or binary data |
| externalReference | attachments | yes | string | if externalSystem equals to URL, it must be an URL. Otherwise, it will be treated as filename |
| binaryData | attachments | no | string | mandatory if externalSystem equals to binary. It should contain base64 encoded content of file to be sent |
| mimeType | attachments | yes | string | MIME type for attachment |
| requestId | -- | yes | uuid | ID of request being processed, unique within HostingApp |
| source | -- | yes | string | HostingApp Key, assigned by startADAM. |
| method | -- | yes | string | Must be 'messageExchange' |

# Support
We´re here to help! Drop a line to our chat support at https://startadam.com
Or send an email to support@startadam.com
