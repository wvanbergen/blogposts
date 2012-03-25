# Why developers should be force-fed state machines

This post is meant to create more awareness about state machines in the web application developer crowd. If you don’t know what state machines are, please read up on them first. Wikipedia is a good place to start, as always.

## State machines are awesome

The main reason for using state machines is to help the design process. It is much easier to figure out all the possible edge conditions by drawing out the state machine on paper. This will make sure that your application will have less bugs and less undefined behavior. Also, it clearly defines which parts of the internal state of your object are exposed as external API.

Moreover, state machines have decades of math and CS research behind them about analyzing them, simplifying them, and much more. Once you realize that in management state machines are called business processes, you'll find a wealth of information and tools at your disposal.

## Recognizing the state machine pattern

Most web applications contain several examples of state machines, including accounts and subscriptions, invoices, orders, blog posts, and many more. The problem is that you might not necessarily think of them as state machines while designing your application. Therefore, it is good to have some indicators to recognize them early on. The easiest way is to look at your data model:

- Adding a state or status field to your model is the most obvious sign of a state machine.
- Boolean fields are usually also a good indication, like published, or paid. Also timestamps that can have a NULL value like published_at and paid_at are a usable sign.
- Finally, having records that are only valid for a given period in time, like subscriptions with a start and end date.

When you decide that a state machine is the way to go for your problem at hand, there are many tools available to help you implement it. For Ruby on Rails, we have the excellent gem state_machine which should cover virtually all of your state machine needs.

## Keeping the transition history

Now that you are using state machines for modelling, the next thing you will want to do is keeping track of all the state transitions over time. When you are starting out, you may be only interested in the current state of an object, but at some point the transition history will be an invaluable source of information. It allows you to answer all kinds of questions, like: “How long on average does it take for an account to upgrade?”, “How long does it take to get a draft blog post published?”, or “Which invoices are waiting for an initial payment the longest?”. In short, it gives you great insight on your users' behavior.

When your state machine is acyclic (i.e. it is not possible to return to a previous state) the simplest way to keep track of the transitions is to add a timestamp field for every possible state (e.g. confirmed_at, published_at, paid_at). Simply set these fields to the current time whenever a transition to the given state occurs.

However, it is often possible to revisit the same state multiple times. In that case, simply adding fields to your model won’t do the trick because you will be overwriting them. Instead, add a log table in which all the state transitions will be logged. Fields that you probably want to include are the timestamp, the old state, the new state, and the event that caused the transition.

For Ruby and Rails, Jesse Storimer and I have developed the Ruby gem state_machine-audit_trail to track this history for you. It can be used in unison with the state_machine gem.

## Deleting records?

In some cases, you may be tempted to delete state machine records from your database. However, you should never do this. For accountability and completeness of your history alone, it is a good practice to never delete records. Instead of removing it, add an error state for any reason you would have wanted to delete a record. A spam account? Don’t delete, set to the spam state. A fraudulent order? Don’t delete, set to the fraud state.

This allows you to keep track of these problems over time, like: how many accounts are spam, or how long it takes on average to see that an order is fraudulent.

## In conclusion

Hopefully, reading this text has made you more aware of state machines and you will be applying them more often when developing a web application. Disclaimer: like any technique, state machines can be overused. Developer discretion is advised.
