# [fit] npm registry 2.0
# [fit] deep-dive

---

Until the last couple of months there
was very little node in npm's registry

It was mostly a javascript application running
embedded in couchdb

some support systems written in node

we just finished a big rewrite: now we're a node service

or set of services



---

# the stack

* haproxy for load balancing
* couchdb for package data storage
* postgres for all other data
* redis for caching
* nginx for static files
* node-restify for node services


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
* 'renv': recursively stores & reads json blobs from `etcd`.

---

# configuration via etcd

## https://github.com/coreos/etcd

A highly available key/value store intended for config & service discovery. We

^ How we do configuration.

---

# lots of complexity

* each piece has a well-defined responsibility
* each piece can be redundant
* exceptions: db write primaries

---

# conservatism won with node

* we're mostly on node 0.10.37
* memory leaks
* will try again with iojs 1.8.x
* or with node now that iojs is folded in :)

---

# operational excitement

* We are still discovering the weak points.
* Every incident results in new alerts.
* Metrics are going to matter more to us.
* (This system is at least more observable than couchdb.)

---

# [fit] npm loves you
