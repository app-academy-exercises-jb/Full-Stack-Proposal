# Project: Slack Clone

## Title: YASC (Yet Another Slack Clone)

### <u>Key Feature Set</u>:

0. Hosting on Heroku

1. Authentication Functionality
	- Registration
	- Multiple login
	- Demo login
	- __BONUS:__ implement auth0 login/registration

2. Live Chat
	- Websocket connection to clients
	- Strict conistency model around messages(see atomic broadcast notes)
	- Real-time Messaging Server 

3. Workspace Functionality
	- Workspace creation -> Workspace admins
	- Workspace un/resgistration
	- Slack-like styling

4. Channel Functionality
	- Channel creation -> Channel admins
	- Channel un/subscription
	- Slack-like styling
	- Typing/presence indicators
	- Websocket functionality for client communication, with client storage of last known events
	- __BONUS:__ Client side pub/sub: client subscribes to list of users/channels that they are 'interested in', and only receive real-time notifs from those. this reduces the amount of events clients have to handle. examples include: presence updates
	- __BONUS:__ Lazy Channels: as opposed to loading the full history of a given channel/thread, only load some of it, loading the rest on demand (ie, user scrolling)

5. Direct Messages (p2p, p2group)
	- Users can dm anyone in the same workspace
	- Users can enter/leave dms

6. Authorization Functionality (workspace, channel)
	- Workspace creators are workspace admin
	- Workspace admin may set permissions of other workspace users
	- Workspace users w/ clearance (ie. channel admins) may create channels
	- Channel admins may boot/mute channel users

7. Link Auto-Loading
	- Async page load implemented by job queue

8. Production Readme

9. __BONUS:__ Deal with scalability by approximating a solution to the atomic broadcast problem in a distributed computing scenario. That is, if we were to have multiple servers running our software, and multiple clients connected to these, we would still expect for there to be a single, stable source of truth. The consistency model we will use will approximate Slack's actual solution, which is called eventual consistency. In essence, this is accomplished by separating 

### Atomic Broadcast Notes/Guarantees: 
- If a valid user broadcasts a message to the channel, all valid users will eventually receive it
- If a valid user receives a message, all valid users eventually receive it
- Uniform integrity of messages: a message is received at most once by each valid users, if it was broadcast
- Uniform order of messages: all valid users receive all messages in the same order

### <u>Architectural Design:</u>

**React (on Rails) Client** - connects to:
1. WebApp (Rails Main API)
2. Messaging Server (Rails Messaging API)

**Messaging Server** - connects to:
1. WebApp

**WebApp** - controls
1. PostgreSQL database
2. Job Queue for async actions (BONUS)

### <u>Architectural Considerations</u>

**React Client**: Responsible for rendering all views for the client. Speaks HTTP with WebApp (thru React), websocket with Messaging Server
WebApp: Responsible for implementing all business logic. It keeps records in a PostgreSQL database, and uses a job queue system to execute asynch jobs.
Messaging Server: Uses the WebSocket protocol (ActionCable in our case) to send real-time messages to users. Specifically, it listens for events that happen in the WebApp/db, and then fans those events out to the relevant users

**Login Flow**:
1. User sends HTTP POST with user auth token to WebApp, which responds with a JSON object describing the current state of the world (cache_ts, latest_ts, websocket url). User communicates with the Messaging Server using the timestamps provided by the WebApp, which the Messaging Server uses to update the user appropriately.

**Websocket Events**:
- chat messages
- typing indicators
- user presence changes
- user profile changes
- channel creations

### <u>References:</u>
- [1] Bing Wei's presentation on Slack's scalability considerations:
https://www.infoq.com/presentations/slack-scalability/
- [2] Keith Adam's podcast on Slack's messaging architecture:
https://softwareengineeringdaily.com/wp-content/uploads/2018/11/SED722-Slack-Architecture-2.0.pdf