build-lists: true
# [fit] npm registry 2.0
# [fit] deep-dive

---

![left](images/bumper2_nasa_big.jpg)
# [fit] C J Silverio
## [fit] director of engineering, npm
## [fit] @ceejbot

^ In case you forgot who I am: this is who I am this year. Next year I might be somebody else. YNK.

---

![fit](images/registry_monolith.png)

---

![](images/registry_monolith.png)
# [fit] registry 1.0
# [fit] embedded in couchdb

---

# [fit] javascript but not node

^ some support systems written in node

---

# advantages

* couchdb's replication is awesome
* didn't have to implement auth
* got away with storing package tarballs as couch attachments
* worked for a longer time than we deserved

---

# disadvantages

* all of this fell over at scale
* tarballs fell over first
* we aren't erlang experts

^ Base64-encoded binary blobs. Observability. Couch has a module system, but it isn't npm.  

---

# late 2013: stay up

* pulled out tarballs
* put varnish in front of everything
* fastly CDN for geolocality

---

# early 2014: stability

* tarballs onto a file system
* found & stomped problems with our couchdb installation
* load-balanced everything

^ This was stable enough that 2014 became the year that node exploded. Node wasn't changing; npm worked. You all started using node to do front end development as well as the back end.

---

# [fit] end 2014: rewrite

* future scaling
* ability to add features easily
* a use for our node expertise
* npm's goal is to be self-sustaining

^ In order to have money coming in, we needed to make something worth paying for. we needed to start adding features Adding those features to the app embedded in couch was a non-starter

---

# [fit] shipped the core of it
# [fit] as npm-enterprise
# [fit] "npm in a box" service

---

# [fit] had a working registry in node
# [fit] before we migrated the
# [fit] public registry to it

^ This was a great move, because we had a chance to stomp the big bugs long before we even considered moving the main registry over.

---

# [fit] April 2015
# [fit] registry 2.0 in production

^ I teased this on twitter periodically. I would announce that "you're soaking in it" when new registry was live.

---

# [fit] turning on private modules
# [fit] was flipping a feature switch

^ This is how you want it to be.

---

# [fit] registry 2.0:
# [fit] lots of node
# [fit] yay microservices

^ Now we start diving into details. I love hearing about the details of other companies' stacks, so I'm sharing now in the hopes that you share too. You ready?

---

# the stack (top)

* Fastly as our CDN
* aws ec2
* we do *not* use EBS or other AWS-specific techs
* ubuntu trusty

^ Amazon! Everybody uses it. it's cheap. It give us lots of control. Ubuntu is the least annoying of the linux distros. I'd pick debian if it didn't exist. Mostly DC redundant, with all single pts of failure in us-west-2.

---

# the stack (middle)

* haproxy for load balancing & tls termination
* a couple instances of pound for tls
* nginx for static files
* redis for caching

^ Phasing pound out. Love the other three.

---

# [fit] the dbs

* couchdb for package data storage
* postgres for users, billing, access control lists
* replica of the package data in postres to drive website

---

# [fit] node frameworks

* web site only: hapi
* everything else: restify

^ The public downloads service is hapi, but we'll rewrite that when we get around to making it perform well.

---

# node-restify

* simple
* barely a framework
* observable
* sinatra/express routing
* we like the connect middleware style

---

# configuration via etcd

## https://github.com/coreos/etcd

A highly available key/value store intended for config & service discovery. We recursively store & extract json blobs from it using `renv`.

^ How we do configuration.

---

# [fit] automation via ansible

## [fit] any box can be replaced
## [fit] by running an ansible play

^ We love it.

---

# lots of complexity, but

* each piece has a well-defined responsibility
* each piece can be redundant
* exceptions: db write primaries
* each service can be worked on in isolation

---

# downsides

* yay distributed systems
* backpressure isn't handled well
* some single points of failure: db primaries
* metrics are a work in progress
* everything is hand-rolled

^ We rehearse the firedrill of replacing primaries. We also rehearse restoring from backup.

---

# conservatism won with node

* we're mostly on node 0.10.37
* memory leaks, some networking trouble
* will try again with iojs 1.8.x
* or with node now that iojs took over :)

---

# git deploy

`git push origin branch:deploy-production`

That's it. You've deployed.

---

# A git-deployable service

- provisioned by ansible
- haproxy load-balancing & monitoring
- webhooks server
- github webhooks trigger a bash script
- any server can have many apps git-deployed to it

The components are all open-sourced.

---

# open sourced parts

* `jthooks`: run by ansible to set up github web hooks
* `jthoober`: a webhook server run on each box that listens for webhook pushes from github
* `rderby`: rolling restarts that stop when monitoring endpoints don't respond
* `renv`: recursively stores & reads json blobs from `etcd`.
* `ndm`: generate upstart/whatever scripts from a service.json config

---

![fit](images/scalable_pieces.png)

^ Here's a pretty handwave-y block diagram of the registry. Each of these pieces is a scalable unit.

---

![fit](images/doubled_pieces.png)

^ Hey look, I just made most of my system redundant across AWS, just by replicating the logical modules.

---

![fit](images/registry_plus_web.png)

^ Here's what it looks like in detail. Made this diagram for myself to keep track of everything; some things are missing.

---

# [fit] we're ready for the future
# [fit] install all the modules!

---

# [fit] npm loves you
