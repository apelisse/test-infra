Fetcher retrives a github repository history and stores it in a MySQL local
database.

For now, it downloads three types of resources:
- Issues (including pull-requests)
- Events (associated to issues)
- Comments (regular comments and review comments)

All of these resources will allow us to:
- Compute average time-to-resolution for an issue/pull-request
- Compute time between label creation/removal: lgtm'd, merged
- break-down based on specific flags (size, priority, ...)

The model is described more precisely in [../sql/]().

Fetcher only downloads what is not already in the SQL database by trying to find
the latest events it knows about. It will poll from Github on a regular basis,
and this can be configured with the `--frequency` flag.

There is no set-up required as `fetcher` will create the github database if it
doesn't exist, along with the various required tables.

Design decision
===============

Because of the size of the repository:

- Accessing comments API doesn't work and returns 500 Internal Error all the
  time.
- Accessing random page of events fail quite often

One of the drawback is the following: We could use the comments API to fetch all
the comments through the same URL, but it doesn't work, so comments are
downloaded for each individual issue that is modified. This significantly
increases the number of requests.

Testing locally
===============

It is strongly suggested NOT to download Kubernetes/kubernetes from scratch with
the fetcher, as it puts a lot of pressure on the token and takes a significant
amount of time. It has been done already and one should use an export of the
existing database.

It is fine to download a smaller repository for testing, like
kubernetes/test-infra or kubernetes/contrib.

Deploying the fetcher
=====================

First time only
---------------

Create the github-tokens secret:
```
kubectl create secret generic github-tokens --from-literal=token_login_1=${your_1st_token} --from-literal=token_login_2=${your_2nd_token}
```

And install the certificate if you haven't done it yet:
```
curl -O https://curl.haxx.se/ca/cacert.pem
kubectl create configmap certificates --from-file=ca-certificates.crt=cacert.pem
```

Deployment
----------

Run the deploy script:
```
make deploy IMG=gcr.io/your-repo/fetcher
```
