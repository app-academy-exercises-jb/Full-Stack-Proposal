# Project: Slack Clone

## Title: YASC (Yet another Slack clone)

### <u>Key Feature Set</u>:

0. hosting on heroku
1. authentication functionality
	- registration
	- multiple login
	- demo login
	- BONUS: implement auth0 login/registration
2. live chat
	- websocket connection to clients
	- strict conistency model around messages(see atomic broadcast notes)
	- real-time Messaging Server 
3. workspace functionality
	- workspace creation -> workspace admins
	- workspace un/resgistration
	- slack-like styling
4. channel functionality
	- channel creation -> channel admins
	- channel un/subscription
	- slack-like styling
	- typing/presence indicators
	- websocket functionality for client communication, with client storage of last known events
	- (BONUS) client side pub/sub: client subscribes to list of users/channels that they are 'interested in', and only receive real-time notifs from those. this reduces the amount of events clients have to handle. examples include: presence updates
	- (BONUS) lazy channels: 
5. direct messages (p2p, p2group)
	- users can dm anyone in the same workspace
	- users can enter/leave dms
6. authorization functionality (workspace, channel)
	- workspace creators are their admin
	- workspace admin may set permissions of other workspace users
	- workspace users w/ clearance (ie. channel admins) may create channels
	- channel admins boot/mute channel users
7. link autoloading
	- async fetch implemented by web workers
8. production readme
9. BONUS: deal with scalability by approximating a solution to the atomic broadcast problem in a distributed computing scenario. that is, if we were to have multiple servers running our software, and multiple clients connected to these, we would still expect for there to be a single, stable source of truth. the consistency model we will use will approximate Slack's actual solution, which is called eventual consistency. in essence, this is accomplished by separating 

### Atomic broadcast notes/guarantees: 
- if a valid user broadcasts a message to the channel, all valid users will eventually receive it
- if a valid user receives a message, all valid users eventually receive it
- uniform integrity of messages: a message is received at most once by each valid users, if it was broadcast
- uniform order of messages: all valid users receive all messages in the same order

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

**React Client**: responsible for rendering all views for the client. speaks HTTP with WebApp (thru React), websocket with Messaging Server
WebApp: responsible for implementing all business logic. it keeps records in a postgresql database, and uses a job queue system to execute asynch jobs.
Messaging Server: uses websockets (ActionCable in our case) to send real-time messages to users. specifically, it listens for events that happen in the webapp/db, and then fans those events out to the relevant 

**Login Flow**:
1. User sends HTTP POST with user auth token to WebApp, which responds with a JSON object describing the current state of the world (cache_ts, latest_ts, websocket url). User communicates with the Messaging Server using the timestamps provided by the WebApp, which the Messaging Server uses to update the user apropriately.

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