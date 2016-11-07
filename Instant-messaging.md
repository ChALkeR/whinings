# What should a sane instant messenger look like?

_Note: this is a conjunction._

* Secure.
* Peer to peer, no centralized auth.
* Supports logs and synchronizes them.
* Deleteable logs (one-side deleting, but from all devices).
* End-to-end encryption.
* Three states for contacts — unvalidated (default), validated, compromised.
* Suggest validating contacts (and other means of promoting security).
* Simple contact sharing.
* Messages should be received by all devices at the same time, no blocking and no «priorities».
* Voice and video.
* File transfer.
* Sensible protection from stealing an authorized device (to the extent possible).
* Stealing an authorized device shouldn't make previous traffic decryptable.
* Chat rooms (aka groups), private or public.
* Audio/video multiconference, private or public.
* No self-invented crypto, instead built on existing libraries.
* Cross-platform, work on any devices.
* FLOSS.
* Message delivery should be guaranteed, i.e. if a message was not delivered for some reason — it should be resent.
* Messages should be ordered correctly (in logical order), if Alice sent A first, then B — Bob should see it that way.

## How?

 * No centralized servers.
 * Users should have their private/public keys, stored at each device. Those should be unique per user device, no need to copy them («device» here and below is actually a user profile on a PC, a phone, or any other hardware device).
 * No profile «copying» between devices, they should just acknowledge each other as being the same virtual account and start syncing their state (logs, contacts), and promote each other as an alternative device to other people's contact list.
 * Any message to any user device gets copied to all their devices as soon as they get online.
 * An ability to «untrust» a device from other devices is needed — for removed or stolen devices.
 * No «offline» messages technically, but they are not blocking — Alice could write anything to Bob at any time — it's queued on Alice device, once that Alice device gets online — it's copied to all Alice devices that are online, once any Bob device gets online — it's copied there, once any other Bob devices get online — it's copied there from Alice or Bob devices.
 * Logs are encrypted and signed, and those could be synced between any users that own them. E.g.:
   1. Alice has device A1 online. Bob is offline. Alice sends message to Bob.
   2. Bob device B1 gets online, Alice message is delivered from A1 to B1.
   3. Bob device B2 gets online, Alice message is delivered from A1 or B1 to B2.
   4. Alice device A1 gets offline and A2 gets online, Alice message is delivered from B1 or B2 to A2.
 * Users in contact list have three visual validity states: unvalidated (default), validated, compromised. Those could be changed by regular fingerprint validation though a separate channel (i.e. user action).
 * Adding a user means adding one or several his devices to contacts as one entry. Other devices are added automatically by the known devices.
 * Contacts could be transferred between users, shared as an ID or QR code. Even nameservers is possible that provide `user@example.com` usernames as a mean of sharing an user ID (which actually includes a pubkey), but those are of course marked as «unvalidated».
 * Voice and video, file transfer — stuff like this should be possible once you implement secure routing between clients. Voice and video should not be synced between user devices by default — there is no need for that, but perhaps making it possible would have some benefits.
 * The client may or may not store transmitted files as regular messages, but if it does — it should sync them between clients of one user.
 * Device public key is used only for initializing a session, i.e. session key exchange. Further conversation is encrypted using session keys, which are auto-renewed and get disposed of fast. We do need perfect forward secrecy.
 * For crypto, use [libsodium](https://github.com/jedisct1/libsodium), _and_ make sure you don't do awful stuff like comparing keys with `memcmp` or running `memset`+`free`.
 * Deleting logs is also a state change, but that promotes only to users _own_ devices. It also blacklists those logs so they are not received anymore.
 * Chat rooms also could be done via state synchronization between all members. They could have access permissions enforced by the creator — they should just sign the «permissions state» of the room. More moderators or even admins could be added in such a way.
 * Message order — use a directed acyclic graph to sort messages first, then use time (to choose the best order of possible ones). Sending a new message should try to re-send all messages it depends on, and it should depend on previously visible/sent text messages. Files/audio is a separate situation — they shouldn't block sending further messages, even if they were undelivered.
 * In chat rooms, receiving a message older than the last one could be blocked until at least a certain amount of users confirm it, in a way so that message should be put to the end in the list, according to when it was actually received by the client. Logs should also be received from admins/moderators or a certain amount of regular users to be displayed. This should be done to protect from the situation when a user spoofs their own messages as sent long time ago (e.g. tries to insert them in logs).
 * In _large_ and/or public chat rooms, the above restrictions could be not enough — a huge number of bots could spoof logs. In this situation, for large and/or public rooms, admins/moderators could enable a mode where logs are only trusted when received from admins/moderators or are signed by them. This could be even enabled automatically after the participants count reaches a certain threshold.

## Caveats and TODO

 * The person you are talking to will see how many «devices» do you have and when you switch them. Some could be associated with work/home. Not sure if this an actual issue given that current instant messengers deliberately display when the user uses a phone, also e.g. Jabber has different priorities and clients — they leak the same information. _We could try to solve this with rotation, though._
 * No true «offline» messages by default — those will get delivered only when the sender and the recipient will have at least one device online at the same moment. Given how often people are online now, mobile phones and desktop PCs — this is probably a better choice then queuing the messages elsewhere. Users could work-around that by having an instance running somewhere, or by selecting an undelivered messages server (see below).
 * Voice and video could require some work to reduce lag. Or not.
 * Spam/bot protection in chat rooms is not covered here.

## Optional features

 * Mobile — push notifications are required, so each mobile application will have to have it push notification server to be able to receive messages while the device is sleeping. Those servers are independent from the network, and are operated by mobile application authors. They are not able to see the messages. The mobile app could have a status hint «ping _this_ server when sending messages, if delivery doesn't work without it». The sending device would send a notification the server, specifying the id of the device that needs to receive the notification, and signed by the sending device. The server would issue a push notification to the target device, that would wake up and do stuff. It also should probably apply a heavy rate-limit for push messages from unknown devices.
 * Opt-in temporary undelivered messages servers — servers that store e2e-encrypted messages for a short period of time (a day or a week). The servers can not peek onto the message contents (remember — those are encrypted with end-to-end encryption). Users can opt-in to this feature and select such a server, and by doing so they will notify all their contacts that any undelivered messages should be re-sent (encrypted) to a specific server. When a client goes online, it retrieves the messages for itself from that server. Those servers would also be needed for merging two accounts, only one of which could be online at each point of time (e.g. dualboot without a third device).
 * Nameservers — to register your id as `name@example.org`. Those are independent from the network, and that binding should be signed by the id itself — so that the nameserver couldn't route to an id that has not agreed to be named. It could route to the wrong id, so any accounts retrieved this way should be unvalidated (as the ones retrieved by most other means).
 * Transports — those would be possible only between your own PC and you other clients. So, if a user runs a client on a PC and uses that to login to other networks (could be done through libpurble), they could use such other networks from any other device by routing messages through the PC. This could be useful as a multi-protocol mobile client that doesn't consume too much resources. Note that connections over insecure networks (and almost all of them are insecure without OTR) should be marked such in a clearly visible way in the GUI of both desktop and mobile clients. Transports would require users who use them to keep their home/work PC turned on, but that's the price for not transmitting other networks credentials to thirdparties.

## What does an insane instant messenger look like?

_Note: this is a disjunction._

* Authentication through SMS codes as the only or default method
* Any other reason for binding accounts to mobile phone numbers
* Server-side logs in plaintext
* Priorities for devices, separate delivery — you receive messages at random devices
* No log sync
* No end-to-end encryption
* Has end-to-end encryption or other secure features, but fails to explain users how to use it
* Unsafe defaults
* Broken message delivery (incorrect order, etc).

## What's wrong with …

### Jabber

 * End-to-end encryption (OTR) is optional and is seldom used.
 * A lot of extensions (XEPs), everything is implemented though them. Part of those were never finished, part of those are already dead.
 * As a result — a lot of incompatible clients that don't work with basic features.
 * No long sync usually.
 * When log sync is present — it's done by storing the logs on the server in plaintext, which is unacceptable.
 * Client priorities, their logic, and the lack of log sync result in messages coming to seemingly random places if several devices are used. That's not what users are used to now.
 * p2p mode is possible, but seems to be incompatible with some other basic features (multi-client, log sync).
 * In federation mode (default) — some servers die, some get turned off for some time for maintenance. Users are forced to wait or even switch to another servers, which involves notifying their whole contact list of the address change. 
 * Audio/Video is still more broken than not.

### Telegram

 * Poses itself as very secure in its PR and on their homepage, but is insecure by default.
 * Server-side logs in plaintext and no end-to-end encryption for everything but secret chats.
 * Security is mentioned multiple times on their homepage. Secret chats aren't. Web client that doesn't have them — is.
 * Attaches user profiles to mobile phones, and those are linked to passports or other ids in some countries.
 * «2FA» is disabled by default — default auth is through SMS codes only.
 * Even in «secret chats», it doesn't hint anyone that they should actually _validate_ the key manually. Instead, that option is hidden deep in the GUI. Without that, «secret» chats are pretty pointless.
 * Stores your contacts on their servers, in plaintext.
 * Note that your contacts means your phonebook, Telegram gets access to it unless you chose to opt-out. _(Last time I checked, opting out on iOS meant that Telegram contacts did not function at all)._
 * Incorrect message delivery order in case of connection issues.
 * Server-side is proprietary, no source code, not possible to run an independent network.
 * Because of previous point — no guarantees, the whole chat network could be shut down or restricted at any time.

### Slack

 * No end-to-end encryption.
 * Logs and saves everything on server-side. Server-side could do anything with your data.
 * Proprietary, no source code.
 * Logs above 10k messages per team are paid, starting from $6.67 per user per month.
 * Server-side is also proprietary, no source code, not possible to run an independent network.
 * Because of previous point — no guarantees, the whole chat network could be shut down or restricted at any time.

### Gitter

 * No end-to-end encryption.
 * Logs and saves everything on server-side. Server-side could do anything with your data.
 * Private rooms with more than 25 users are paid and cost $5 per user per month.
 * Incorrect message delivery order in case of connection issues.
 * Proprietary, no source code for the client.
 * Server-side is also proprietary, no source code, not possible to run an independent network.
 * Because of previous point — no guarantees, the whole chat network could be shut down or restricted at any time.

### Tox

 * No log sync.
 * Lacks basic features, doesn't seem to work reliably.
 * Some incorrect design decisions, like adding another account requires copying the complete profile box to it, which is undesirable (you better don't export your profile to a usb flash disk for than and give it to your friend/colleague later, even after deleting the file).
 * Crappy code. They even had `memset` + `free` and compared keys with `memcmp` in several places. Ref: [irungentoo/toxcore#1506](https://github.com/irungentoo/toxcore/pull/1506), but it looks like there are still more bad things happening there. 

### Irc

 * No end-to-end encryption. Server-side could do anything with your data.
 * No log sync.
 * No offline messages without a bouncer, bouncers are insecure — they have your password if you need private messages for you account, and store logs on a thirdparty server (which VPS usually is).

### Skype

 * No end-to-end encryption.
 * Logs and saves everything on server-side. Server-side could do anything with your data.
 * Reported as being a spyware several times — it was recursively reading browser profiles, was trying to read motherboard info, etc.
 * Proprietary, no source code for the client.
 * Server-side is also proprietary, no source code, not possible to run an independent network.
 * Because of previous point — no guarantees, the whole chat network could be shut down or restricted at any time.

### VK

 * No end-to-end encryption.
 * Logs and saves everything on server-side. Server-side could do anything with your data.
 * Incorrect message delivery order in case of connection issues.
 * Proprietary, no source code for the client.
 * Server-side is also proprietary, no source code, not possible to run an independent network.
 * Because of previous point — no guarantees, the whole chat network could be shut down or restricted at any time.

### Facebook

 * No end-to-end encryption.
 * Logs and saves everything on server-side. Server-side could do anything with your data.
 * Proprietary, no source code for the client.
 * Server-side is also proprietary, no source code, not possible to run an independent network.
 * Because of previous point — no guarantees, the whole chat network could be shut down or restricted at any time.

### Pidgin, Empathy, KDE-Telepathy, Chatsecure, Xabber, QIP...
 * See [Jabber](#jabber) (or any other protocol that you are actually using).

### Ring

 TODO

### Retroshare

 TODO

### Bleep

 TODO

### Hangouts

 TODO

### Signal

 TODO

### Snapchat

 TODO

### iMessage

 TODO

### Viber

 TODO

### Mattermost

 TODO
