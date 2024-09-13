<!--
.. title: Designing a Chat Application
.. slug: designing-a-chat-application
.. date: 2024-09-13 18:54:30 UTC+05:30
.. tags: system-design
.. category: 
.. link: 
.. description: 
.. type: text
-->

# Overview and Requirements

We are going to design a chat application that will allow users to send messages to each other. The chat application should support the following requirements:

1. One to One Chat: Users should be able to send messages to other users.
2. Group Chat: Users should be able to create groups and send messages to all group members.
3. Push Notifications: Users should be able to receive push notifications for new messages.
4. Online Status: Users should be able to see the online status of other users.
5. Read Receipts: Users should be able to see if their messages have been read by the recipients.

<!-- TEASER_END -->

# Design Diagram

![Messaging Service](/images/chat_design.png)

Here is a brief description of the components:

* **Client Application**: The client application is used by users to send and receive messages. It communicates with the messaging service to send and receive messages.
* **Gateway Service**: The gateway service is responsible for routing messages to the appropriate service. It maintains a WebSocket connection with the client application for real-time communication.
* **Session / Chat Service**: Our Main service for handling chat messages. It is responsible for storing and retrieving messages, managing user sessions. It has it's own key-value store to store messages and user sessions.
* **Group Service**: The group service is responsible for managing groups. It allows users to create groups, add members to groups, and send messages to groups. Size of group is limit to a fixed number.
* **Presence Service**: The presence service is responsible for managing the online status of users. It keeps track of which users are online and which users are offline. Every time an event is sent to the gateway service from an user, it updates the presence service.
* **Push Notification Service**: The push notification service is responsible for sending push notifications to users when they receive a new message. Session Service or Group Service will invoke this service if the user is offline.


# Application Flows

## User Login

* When a user logs in, the client application sends a login request to the Gateway Service.
* The Gateway Service requests all pending messages from the Session Service and registers a session.
* The Session Service retrieves all pending messages for the user and sends them to the Gateway Service.

## One to One Chat

* Client sends a message to the Gateway Service with user_id of the recipient and the message.
* Gateway Service forwards the message to the Session Service.
* Session Service stores the message and sends it to the gateway service connected to the recipient. This information is stored in the key-value store.
* If online, Gateway Service sends the message to the recipient's client application.
* Otherwise Gateway Service sends the message to the Push Notification Service.
* On Read, the recipient sends an event to the Gateway Service, which updates the message status in the Session Service.

## Group Chat

* Client sends a message to the Gateway Service with group_id and the message.
* Gateway Service forwards the message to the Group Service.
* Group Service stores the message and sends it to all members of the group.
* This operation is expensive as it involves sending the message to multiple clients. That is why we have a limit on the group size.

## Online Status

* When a user makes an event, the Gateway Service sends an event to the Presence Service to update the online status of the user.
* The Gateway Service sends an event to the Presence Service to update the online status of the user.
* The Presence Service updates the online status of the user.
* An offset is used to mark a user offline if they have not made an event in a certain amount of time.

## Push Notifications

Notifications are sent to users when they are offline. The Push Notification Service is invoked by the Session Service or Group Service when a user is offline. Following Third Party services can be used for Push Notifications:

* Firebase Cloud Messaging (FCM)
* Apple Notification Service (APN)

# Data Models

## Message

We define the field `eventType` to distinguish between user activity and application events.

### Group Message
```json
{
    "message_id": "123",
    "sender_id": "123",
    "recipient_id": "456",
    "group_id": "123",
    "message": "Hello",
    "status": "sent",
    "eventType": "user",
    "timestamp": "2024-09-13T18:54:30"
}
```

### One to One Message
```json
{
    "message_id": "123",
    "sender_id": "123",
    "recipient_id": "456",
    "message": "Hello",
    "status": "sent",
    "eventType": "user",
    "timestamp": "2024-09-13T18:54:30"
}
```

## User

```json
{
    "user_id": "123",
    "name": "Alice",
    "email": "alice@gmail.com",
    "password": "hashed_password"
}
```

## Group

```json
{
    "group_id": "123",
    "name": "Group 1",
    "members": ["123", "456", "789"]
}
```

## Presence

```json
{
    "user_id": "123",
    "online": true,
    "last_active": "2024-09-13T18:54:30"
}
```

# Conclusion

The current design of the system satisfies our requirements. We have a scalable and fault-tolerant system that can handle a large number of users. There are a number of functionalities that can be added like file sharing, video calls. An important point here is that we are storing the user messages. The persistance requirement may not may not be required depending on the business model.
