--- 
title: IPFS + ENS Integration
sample: Open Source Contribution
date: 2020-12-28 12:00:00 +07:00
category: Blockchain
tags: [distributed systems, smart contracts, design, ipfs, ens, blockchain, app, ethereum, open source]
description: Integrating blockchain services IPFS and ENS(Ethereum Name Service). The process of working on a large pull request in a big open source project. 
---

# IPFS & Ethereum Name Service Integration
Most of my professional work ends up not being open source and I think thats probably true for a lot of developers. I'm ok with that and can still contribute to open source projects in my spare time, but it was cool when I got a chance to spend some work time working on an open source contrition. I implemented the [ipfs](https://www.ipfs.io) integration for [ethereum name service](https://ens.domains).

You can see the final in action by using the [ens web app](https://ens.domains).

Featured in the following articles:
- [The Ethereum Name Service](https://medium.com/the-ethereum-name-service/upload-to-ipfs-directly-from-the-ens-manager-with-new-tool-ac055db5d2fe) 
- [Crypto Slate Article](https://cryptoslate.com/temporal-and-ens-now-make-launching-a-decentralized-website-easier-than-ever/)
- [Decrypt Article](https://decrypt.co/26246/ens-unveils-easier-way-to-build-decentralized-websites-on-ethereum)
- [Temporal Announcement](https://medium.com/temporal-cloud/temporal-ens-launch-a-decentralized-website-in-3-clicks-from-the-ens-manager-cb7012bae8fc)

## IPFS 
IPFS is a low level peer to peer hypermedia protocol. It provides similar services to http but does so in a distributed and decentralized fashion. IPFS allows users to not only receive but host content, in a similar manner to BitTorrent. As opposed to a centrally located server, IPFS is built around a decentralized system of user-operators who hold a portion of the overall data, creating a resilient system of file storage and sharing. Any user in the network can serve a file by its content address, and other peers in the network can find and request that content from any node who has it using a distributed hash table (DHT).

## ENS
Ethereum name service or ENS for short is a service that allows anyone to access information in a decentralized manner while still using human readable names. Despite it's name it can be used to turn any cryptographic hash into a human readable name like `name.eth` instead of `7E889AC0EBB9...`. It's name comes because it is built on top of smart contracts, which reside on the Ethereum blockchain. 

## ENS + IPFS
So what does ENS have to do with IPFS? Well IPFS identifies content by a unique cryptographic hash. These hashes are great from a technical standpoint since they allow for a lot of cool things like content addressing, deduplication, etc but they are a pain for users to type in. Saying hey go to `7E889AC0EBB92DB5C487` doesn't roll off the tongue. To get around this you can use ENS to register a human readable name like `test.eth` and the link it to your IPFS content hash. Then you can tell people to go to `test.eth` and they can see your content, whether it's a website, video, pdf, etc.

## Integration
Before the integration you could still link a IPFS hash on ENS but it was a lot more involved. The goal of the integration was to allow people to host decentralized content without having to install anything or deal with low level technical protocols.

It was quite an undertaking but 59 comments later you can see the [merged pull request here](https://github.com/ensdomains/ens-app/pull/581). The reason for a lot of the discussion was that we weren't just connecting ENS and IPFS but we were also building an underlying platform for multiple solution providers to plug into and offer ipfs services. In addition to this I had to design the UX from scratch and integrate the design into the current design of the app. To complicate things further the ENS group was also doing a big rewrite of the app and updating the coding style and some of other components of the app at the same time. This involved additional rewrites on my part but overall made the code base much cleaner.

Overall the ENS group was great to work with though, even though they had a lot going on with their rewrite and upcoming release they still made time to answer any questions, do code reviews, and discuss the overall design.

## Technical Implementation
I'm not going to go into full details but just discuss the overall process. For more specific details you can look through the pull request. I might add more details in the future as I think of them.

### Environment
Since this is a decentralized app getting the development enviornment set up is a lot more involved to get set up than a traditional app even with the help of tools like docker.

You use [ganache](https://github.com/trufflesuite/ganache-cli) to test the blockchain backend. Then you can run a seed script and finally be able to do testing with [Cypress](https://www.cypress.io/).

Also, you have to set up [Graph Node](https://github.com/graphprotocol/graph-node) a Rust service that event sources the Ethereum blockchain to deterministically update a data store that can be queried via the GraphQL endpoint then deploy the smart contracts and deploy the ens subgraph.

### Getting familiar with the app
This is always tricky when joining or contributing to a new project. You don't want to reimplement things that are already there. Also, you have to make sure you style/structure your code following their conventions and style guide(if they have one). For me I even though I was building a new component there were still several integration points and I reused the preexisting styles, buttons, form elements.

### Architecture
There are multiple parts of this integration:
1. Platform layer   
One of the requirements was to make it extensible so that any provider offering IPFS services can make use of it. It starts out with Temporal integrated but we make it easy for other providers to add their information and then the user can select what service they want to use.
2. Login layer   
An integrated login/registration layer. This allows users to verify their platform accounts and store files on IPFS without having to leave the app.
3. File Upload   
This is formatted like a normal file upload you might find in google drive, dropbox, etc. The user can upload a single file or a directory. This is really the only action the user has to do and it gets automatically converted to IPFS structure, uploaded, and hosted. 
4. Convert file structure for IPFS upload   
You can't use the uploaded file with the ipfs api. You have to rebuild the paths but we do all this automatically under the hood.
5. Preview   
After uploading you can preview how your website/file looks before you save the hash to your ENS domain
6. Link with ENS   
Update all the things on ENS's side with the new IPFS hash

As you can see there's a lot of moving parts. Some are traditional web app things you might find in any application like the login & registration. Some are a bit more complicated like the platform layer and file upload but you would still likely find these components in a traditional web app. Lastly you have things related to dApps like IPFS file uploading, working with the ipfs protocol, updating smart contracts, and working with the Ethereum blockchain. 

### Code samples
In the future I'll add specific code samples from the pull request and go over why I did things a certain way. For now you can always view the full pull request [here](https://github.com/ensdomains/ens-app/pull/581).

Some things I would like to cover in the future.
- Integrating the component into the existing app
- Handling the ipfs upload
- Some of the styling/ux choices

## Summary
Some Closing thoughts. 

ENS is a cool project and the group behind it have been great to work with. Definitely check it out! Building a name service is hard by itself, doing it all in a distributed, decentralized way is even harder and they have do a great job

I enjoyed working on this even though a lot of it was javascript. I usually work in a lower level language like C, Rust, or Go but it was nice moving up the stack and to brush of my js skills. I pride myself in being able to work on the fullstack in multiple languages from low level protocol implementations to frontend design and javascript work.

Being able to work on opensource stuff is cool for a couple of reasons. 1) There's a lot of cool opensource projects out there. 2) You can actually talk about your work and point out concrete examples that people can look at.

As always its nice seeing the finished product implemented in production. After going through the process of conception -> design -> implementation -> review -> production and all the steps in-between you can finally see users interacting with the thing you made. 

On that note being there through the whole process is something I've come to like too. When I first started out as a developer I would often just be responsible for one part of the project (mostly implementation & review) but as I got more experienced I've been able to take a larger role and am now comfortable in managing the whole process.

