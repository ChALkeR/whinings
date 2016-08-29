# What should a sane instant messanger look like?

_Note: this is a conjunction._

* Secure
* Peer to peer, no centralized auth
* Supports logs and syncrhonizes them
* Deleteable logs (one-side deleting, but from all devices)
* End-to-end encryption
* Three states for contacts — unvalidated (default), validated, compromised
* Suggest validating contacts (and other means of promoting security)
* Simple contact sharing
* Messages should be received by all devices at the same time, no blocking and no «priorities»
* Voice and video
* File transfer
* Sensible protection from stealing an authorized device (to the extent possible)
* Stealing an authorized device shouldn't make previous traffic decryptable
* Chat rooms (aka groups), private or public
* Audio/video multiconference, private or public
* No self-invented crypto, instead built on existing libraries
* Cross-platform, work on any devices
* FLOSS

## How?

 * No centralized servers.
 * Users should have their private/public keys, stored at each device. Those should be unique per user device, no need to copy them («device» here and below is actually a user profile on a PC, a phone, or any other hardware device).
 * No profile «copying» between devices, they should just acknowledge each other as being the same virtual account and start syncing their state (logs, contacts), and promote each other as an alternative device to other people's contact list.
 * Any message to any user device gets copied to all their devices as soon as they get online.
 * An ability to «untrust» a device from other devices is needed — for removed or stolen devices.
 * No «offline» messages technically, but they are not blocking — Alice could write anything to Bob at any time — it's queued on Alice device, once that Alice device gets online — it's copied to all Alice devices that are online, once any Bob device gets online — it's copied there, once any other Bob devices get onlined — it's copied there from Alice or Bob devices.
 * Logs are encrypted and signed, and those could be synced between any users that own them. E.g.:
   1. Alice has device A1 online. Bob is offline. Alice sends message to Bob.
   2. Bob device B1 gets online, Alice message is delivered from A1 to B1.
   3. Bob device B2 gets online, Alice message is delivered from A1 or B1 to B2.
   4. Alice device A1 gets offline and A2 gets online, Alice message is delivered from B1 or B2 to A2.
 * Users in contact list have three visual validity states: unvalidated (default), validated, compromised. Those could be changed by regular fingerprint validation though a separate channel (i.e. user action).
 * Adding a user means adding one or several his devices to contacts as one entry. Other devices are added automatically by the known devices.
 * Contacts could be transfered between users, shared as an ID or QR code. Even nameservers is possible that provide `user@example.com` usernames as a mean of sharing an user ID (which actually includes a pubkey), but those are of course marked as «unvalidated».
 * Voice and video, file transfer — stuff like this should be possible once you implement secure routing between clients. Voice and video should not be synced between user devices by default — there is no need for that, but perhaps making it possible would have some benefits.
 * The client may or may not store transmitted files as regular messages, but if it does — it should sync them between clients of one user.
 * Device public key is used only for initalizing a session, i.e. session key exchange. Further conversation is encrypted using session keys, which are auto-renewed and get disposed of fast. We do need perfect forward secrecy.
 * For crypto, use [libsodium](https://github.com/jedisct1/libsodium), _and_ make sure you don't do awful stuff like comparing keys with `memcmp` or running `memset`+`free`.
 * Deleting logs is also a state change, but that promotes only to users _own_ devices. It also blacklists those logs so they are not received anymore.

## Caveats and TODO

 * The person you are talking to will see how many «devices» do you have and when you switch them. Some could be associated with work/home. Not sure if this an actual issue given that current instant messengers deliberately display when the user uses a phone, also e.g. Jabber has different priorities and clients — they leak the same information. _We could try to solve this with rotation, though._
 * No true «offline» messages — those will get delivered only when the sender and the recepient will have at least one device online at the same moment. Given how often people are online now, mobile phones and desktop PCs — this is probably a better choice then queing the messages elsewhere. Users could work-around that by having an instance running somewhere. Another way would be to introduce optional log servers, that will keep undelivered messages for a short period of time (day/week), without being able to look into them (remember — those are encrypted with end-to-end encryption). Not sure that is needed, though.
 * Voice and video could require some work to reduce lags. Or not.
 * Chat rooms are not yet covered here in details, but they also should be done via state synchronization between all members.

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

## What's wrong with …

### Jabber

### Telegram

### Slack

### Gitter

### Tox

### Irc

### Skype

