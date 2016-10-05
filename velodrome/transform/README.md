Transform is used to create relevant metrics from the data saved in the SQL
database. The data in the SQL database has the issues and the list of events,
but we may want to have more fancy metrics based on when labels have been
applied, and what happened to the pull-request or issues through it's lifetime.

This logic is written as "Plugins". A quick look at the code ([plugins.go]())
explains the interface of a plugin, and the type of parameters it receives.

Plugins are constantly waiting for:
- Changes to issues (they come by order of modification)
- Events and comments: they come sorted by creation date.

Note that you will always receives changes to issues before receiving events or
comments.

The program periodically fetches from the SQL database to find changes and
pushes them to each plugin.

Walk-through: Creating a new plugin
-----------------------------------

To create a plugin, you need to implement the interface defined in
[plugins.go](), and also register its creation there.

Let's have a look at [merged.go]():

- At registration time, `NewMergedPlugin` tries to get the last measurement in
  the database, so that it doesn't have to re-send what is already there. The
  reason is that we will receive all the events of the repository ever. Including
  some that we may have processed before already.

- We need to implement `ReceiveIssue`, `ReceiveComment`, and `ReceiveIssueEvent`
  even if we don't use all of them.

- When we receive an Event, in this situation, because we want to count how many
  "merged" event we received, we just discard each event that is not
  "merged". Then we discard event that are already in the database because they
  happened before the last event we found. And finally, if none of this
  condition happened, we just insert the value in the database through the
  InfluxDatabase interface.

- We make sure the plugin is registered in [plugins.go]() or it's never going to
  receive any event.

Deploying Transform
===================

Make sure the parameters are fine and matching the configuration of your grafana
stack [../grafana-stack](), and then:
```
kubectl apply -f deployment.yaml
```
