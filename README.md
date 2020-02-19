# Project Proposal: Slack Clone

## YASC (Yet Another Slack Clone)

### <u>Key Feature Set</u>:

0. Hosting on Heroku

1. Authentication Functionality
	- Registration
	- Multiple login
	- Demo login
	- __BONUS:__ Implement Auth0 login/registration functionality

2. Live Chat
	- Websocket connection to clients
	- Strict consistency model around messages (see [Atomic Broadcast notes](#atomic-broadcast-guarantees)). 
      - Because the non-scaled version of this app is an essentially synchronous system (as there is only one server running and receiving messages), there is no need to agree on order or integrity: it is all determined by the single source of truth which is the singular server.
      - This means that implementing Atomic Broadcast should be trivial, as opposed to impossible. 
	- Real-time Messaging Server 
      - We are trying to copy Slack's way of doing things. As mentioned to in the architectural design [section](#architectural-design), there is a messaging bus which is responsible for real-time communications. 
      - In a distributed version of this app, the WebApp would be responsible for spawning many Messaging Servers (sharding them, as it were), and distributing those urls to the appropriate users.
      - The present version of this app will have this bus living on the same server as the WebApp, but at another API endpoint. 

3. Workspace Functionality
	- Workspace creation -> Workspace admins
	- Workspace un/registration
	- Slack-like styling

4. Channel Functionality
	- Channel creation -> Channel admins
	- Channel un/subscription
	- Slack-like styling
	- Typing/presence indicators
	- Websocket functionality for client communication, with client storage of last known events
	- __BONUS:__ Client side pub/sub: client subscribes to list of users/channels that they are 'interested in', and only receive real-time notifications from those. this reduces the amount of events clients have to handle. examples include: presence updates
	- __BONUS:__ Lazy Channels: as opposed to loading the full history of a given channel/thread, only load some of it, loading the rest on demand (ie, user scrolling)

5. Direct Messages (p2p, p2group)
	- Users can dm anyone in the same workspace
	- Users can enter/leave dms

6. Authorization Functionality (workspace, channel)
	- Workspace creators are workspace admin
	- Workspace admin may set permissions of other workspace users
	- Workspace users w/ clearance (ie. channel admins) may create channels
	- Channel admins may boot/mute channel users

7. Production Readme

8. __BONUS:__ Asynchronous Job Queue
	- Async loading of links shared in chat

9. __BONUS:__ Deal with scalability in a distributed computing scenario by approximating a solution to the Atomic Broadcast problem. That is, if we were to have multiple servers running our software, and multiple clients connected to these, we would still expect for there to be a single, stable source of truth. The consistency model we will use will approximate Slack's actual solution, which is called eventual consistency. In essence, this is accomplished by separating 

### <u>Architectural Design:</u>

**React (on Rails) Client** - connects to:
1. WebApp (Rails Main API)
2. Messaging Server (Rails Messaging API)

**Messaging Server** - connects to:
1. WebApp

**WebApp** - controls
1. PostgreSQL database
2. __BONUS:__ Job Queue for async actions

This isn't much more than an attempt to clone Slack's architecture as presented in reference [1], minute 2:10. 

### <u>Architectural Considerations</u>

**React Client**: Responsible for rendering all views for the client. Speaks HTTP with WebApp (thru React), websocket with Messaging Server

**WebApp**: Responsible for implementing all business logic. It keeps records in a PostgreSQL database, and uses a job queue system to execute async jobs.

**Messaging Server**: Uses the WebSocket protocol (ActionCable in our case) to send real-time messages to users. Specifically, it listens for events that happen in the WebApp/db, and then fans those events out to the relevant users

**Login Flow**:
1. User sends HTTP POST with user auth token to WebApp

2. WebApp responds with a JSON object describing the current state of the world (including cache_ts, latest_ts, and a WebSocket url). 

3. User communicates with the Messaging Server at the WebSocket url, passing the timestamps provided by the WebApp, which the Messaging Server uses to update the user with the appropriate delta.

**Websocket Events**:
- Chat messages
- Typing indicators
- User presence changes
- User profile changes
- Channel creations

### <u>Atomic Broadcast Guarantees:</u>
- If a valid user broadcasts a message to the channel, all valid users will eventually receive it
- If a valid user receives a message, all valid users eventually receive it
- Uniform integrity of messages: a message is received at most once by each valid users, if it was broadcast
- Uniform order of messages: all valid users receive all messages in the same order

For more detail please consult reference [3] below. For direct relation to Slack, please check [2].

### <u>Background:</u>

- Previous experience designing and implementing a WebSocket based messaging and general events server (on NodeJS), which can be found powering the live functionality of this simple, multiplayer-only [game](http://la-pipoette.herokuapp.com/).

### <u>References:</u>
- [1] [Bing Wei's (Slack Distributed Systems Team) presentation on Slack's scalability considerations](https://www.infoq.com/presentations/slack-scalability/)
- [2] [Keith Adam's (Slack Chief Architect) podcast on Slack's messaging architecture](https://softwareengineeringdaily.com/wp-content/uploads/2018/11/SED722-Slack-Architecture-2.0.pdf)
- [3] [Wikipedia Atomic Broadcast](https://en.wikipedia.org/wiki/Atomic_broadcast)