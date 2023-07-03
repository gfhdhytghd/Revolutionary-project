rchitecture
## User local
The overall design refers to p2p protocol and icq and other distributed protocols, but does not set a central server, the design to the ultimate anti-surveillance and privacy protection as the goal, the front-end does not consider user friendliness and UI design, can be made into a command line
### Data
#### User ID and nickname
#### Your own public and private keys
#### News
### Key generation
#### RSA
##### 8192 digits
###### Generates the public and private keys
* Public key signature and public key MD5
### out-of-app key storage
#### string
##### Set a password
##### Use this password to encrypt the RSA private key (symmetric encryption) and attach the plaintext MD5
##### Converts the encrypted private key to base-32, generating a string of letters and numbers, and gives the user a request to print or copy by hand or store it in a safe place
##### The application can be read and generated again on normal login, but with the password verified
#### QR code
##### Set a password
##### Use this password to encrypt the RSA private key (symmetric encryption) and attach the plaintext MD5
##### Makes the encrypted private key into three two-dimensional codes, each of which contains the sequence information and part of the key
##### requires users to print a QR code or save it in a trusted location
##### The application can be read and generated again on normal login, but with the password verified
#### Dongle
##### Replace the password entered by the user with the dongle key or both. The remaining steps are the same
#### file
##### also asks for a password, then stores the file directly locally, and asks users to keep the file safe
### Login authentication
#### Enter a string or scan the code or select a file
#### Verification format
#### Enter your password or/and insert the dongle
#### Decryption
#### Verify MD5
### Send
#### Adds time stamps, random check codes (to prevent duplicate messages in some cases), RSA public key encryption, and MD5 operations
#### Check connection (handshake)
##### Handshake unsuccessful
##### Gets the connection from the tracker
###### The peer party is offline or the status is unknown
###### Offline: stored locally for sending
###### uncertain: Send one copy and store one copy until it is confirmed to be online
#### Successful handshake
#### Sends MD5 in ciphertext and plaintext
### Receive
#### Successful handshake
#### Receives the ciphertext and MD5
#### Decrypts the RSA private key and verifies the MD5 authentication
##### MD5 correct
###### As if nothing had happened
##### MD5 is incorrect
###### resend the public key and request resending the message
###### Times over the limit: "Serious connection error"
#### Checks the verification codes and time stamps to eliminate duplicate messages and sort the messages in order based on the time stamps
### Group chat
#### Jiaqun
##### search
##### Verification (optional)
##### Join
#### News
##### 1-50 people
###### similar to the Gossip protocol
###### send
* Determine the online status and obtain the online status table
* Create a routing table
* fan-out=10 (Normal number of online users)
fan-out=50 (small number of people online)
* All but the last level must be online members
* Except for the first level, each level receives at least three transmissions from the upper level
* If there are inactive members at the next level, the inactive member id and the timestamp of the corresponding message are stored in the "To be sent Information table".
* The last level can contain non-online members, broadcast the whole group when online, and then retrieve the "to send message table" and send the unsent message
* Add timestamp (ms level exact value), send route table, random check code, sender ID, next-level member RSA public key encryption and add plaintext MD5
* Sends the MD5 of ciphertext and plaintext to the next level
* The next level receives the message
* RSA private key decryption and MD5 authentication
* MD5 correct
* As if nothing happened
* MD5 is incorrect
* Resend the public key and request resend
* Sort and display messages by timestamp, and display avatars and nicknames by ID
* Read the routing table
* Determine whether to send again
* No
* Yes
* Determine whether all members of the next level are online
* Yes
* No
* Online Members
* Not an online member
* Put the other party ID and message timestamp
Stored in the message to be sent table
* Received the call online
##### 51-2500 people
###### similar to the gossip protocol
###### Randomly select the number of users that is the quadratic root of the total number of users, let's call them zero-level members
* Determine online status
* Some not online
* Exclude the ones that are not online and re-randomly pick them until they are all online. If more than 10 rounds are still not all online, the online status of the whole group is directly obtained, the online people are directly selected, and if the number is not reached, the next step is directly forced
* All online
* Continue
###### Create the sending roadmap
* Divide the remaining users
* Number of tables = number of level zero members
* Each user is included in at least 3 tables
* Create a routing table in each table
* fan-out=10 (Normal number of online users)
fan-out=50 (small number of people online)
(non-final level only)
* All but the last level must be online members
* Except for the first level, each level receives at least three transmissions from the upper level
* If there are inactive members at the next level, the inactive member id and the timestamp of the corresponding message are stored in the "To be sent Information table".
* The last level can contain non-online members, broadcast the whole group when online, and then retrieve the "to send message table" and send the unsent message
###### Adds a timestamp (ms level exact), a routing table for each level zero member, a random check code, sender ID, RSA public key encryption for the level zero member, and MD5 in plain text
###### The message is distributed to each level zero member
###### The zero-level member distributes messages within the table
* Add timestamp (ms level exact value), send route table, random check code, sender ID, next-level member RSA public key encryption and add plaintext MD5
* Sends the MD5 of ciphertext and plaintext to the next level
* The next level receives the message
* RSA private key decryption and MD5 authentication
* MD5 correct
* As if nothing happened
* MD5 is incorrect
* Resend the public key and request resend
* Arrange and display messages by timestamp
* Read the routing table
* Determine whether to send again
* No
* Yes
* Determine whether all members of the next level are online
* Yes
* No
* Online Members
* Not an online member
* Put the other party ID and message timestamp
Stored in the message to be sent table
* Received the call online
#### Group Management
##### gag
##### withdraw
##### Kick out of the group chat
### Advanced authentication
#### Users set their own password
#### Send verification when identity is in doubt
#### Digest Authentication
### tracker get
#### is pushed with version updates
#### The user manually added
### Add friends
#### General
##### Discovered
###### Connect the tracker to query the ID
###### Get a connection
###### Connect to each other, get nicknames, avatars
##### send
###### Send ID, nickname, avatar and
Public key of the signature and MD5 of the public key
##### accepted
###### stores the ID and public key of the other party
###### tracker queries to establish a connection
###### Sends the signed public key and MD5
#### Higher security
##### Encryption key
###### Indicates MD5 of the password
##### Discovered
###### ibid.
##### send
###### The information is the same as above, but the public key is encrypted with AES
##### accepted
###### Enter the password and decrypt the public key
#### no tracker
##### Manually add files
##### File contents
###### ID nickname, profile picture
###### Public key
###### IP (public network or Intranet penetration)
* This item can be updated manually
##### add
###### Detects and establishes a connection
* Send your own information and public key
#### with verification questions
##### Receives user information from the server and encrypted contact details and signature verification questions in plain text
##### Answers the questions and performs MD5
##### Use MD5 to decrypt the contact information
### Online status query
#### Status check
##### Send test information (random number)
##### Calculates and returns (random number + public key, MD5)
##### verification
###### Normal response
* Decision online
###### No response or error
* Connect tracker updates the connection mode and online status
* Connection mode updated, online
* Reexecutes the status check
* Connection mode updated, offline
* Reexecutes the status check
* Pass
* Online
* Not passed
* Offline
* Connection mode is not updated, offline
* Offline
* Connection mode is not updated, online
* "Status unknown"
#### Status check 2
##### Gossip The online status of the spread
#### Test opportunity
##### Global and user information update points
##### Manual
### Information update point determination
#### User
##### When the user chat screen is displayed
##### When sending a message
#### global
##### When you enter the application
##### every once in a while (user set) (front desk only)
### Status update
#### Connection mode
##### System port receives network status changes, detects the network when the status changes, sends a new connection to the tracker, and refreshes it when the PPP connection is unavailable
#### Online status
##### Sends updates to tracker when manually offline
##### Send updates to the tracker and gossip when online
#### Nickname profile picture update
##### update when Gossip spread (the content of the spread is "ID: xx profile picture update", not directly spread the profile picture)
##### Transmits MD5 of the profile picture nickname when entering the chat screen
###### consistent
* Regardless
###### inconsistent
* PPP transfer update
#### Log in to the switch device
### Content review
#### None
### In-app firewall
#### This section is used to prevent leakage caused by third-party modification of the client
#### prevents the application itself from contacting non-user and tracker urls /IP
##### Retrieves IP and connection lists when communicating
#### prevents information tracking servers masquerading as trackers or users
##### Check the format and quantity of traffic
#### This section of the requirement can not be easily removed or modified by some metaphysical way to make the entire core function of the application can not run without this thing
### http camouflage
#### refer to v2ray to disguise the traffic in http form and simulate the behavior of the http server as much as possible
### Manual update of keys
#### Users can manually update their keys if they feel that their keys have been leaked
#### Enter a chat screen and send a public key to a chat partner
## tracker
### Design objectives: Can be simple and fast deployment (x-ui deployment difficulty) without storing private information or information that may cause serious privacy leakage (chat logs, etc.), store as few files as possible, and use cpu and bandwidth as low as possible (requiring that the information stored locally does not exceed 20GB and the bandwidth does not exceed 3mbps when serving 1000w users). With a simple firewall (or a recommended firewall software once deployed), it can withstand a certain level of attack, and the information cannot be directly viewed by the server owner.
### Stored information: (symmetric encryption)
The user ID to connect to here
Online status and communication methods
And the verification problem
#### Encryption mode
##### bitlocker information with a key that is encoded when the software is written. The key is not stored in an open source library. It is encrypted with bytecode in the software
### User query
#### receives commands from users
#### Send the information required by the instruction
### Status determination
#### Receives status updates from the client
### Connection information updated
####, and then the user will connect to the tracker and send the connection information
