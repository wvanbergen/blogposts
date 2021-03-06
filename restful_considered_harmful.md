# RESTful thinking considered harmful

It has been interesting and at times amusing to watch the last couple of intense debates in the Rails community. Of particular interest to me are the two topics that relate to RESTful design that ended up on the Rails blog itself: [using the PATCH HTTP method for updates](http://weblog.rubyonrails.org/2012/2/25/edge-rails-patch-is-the-new-primary-http-method-for-updates/) and [protecting attribute mass-assignment in the controller vs. in the model](http://weblog.rubyonrails.org/2012/3/21/strong-parameters/).

## REST and CRUD

These discussions are interesting because they are both about the update part of the CRUD model. PATCH deals with updates directly, and most problems with mass-assignment occur with updates, not with creation of resources. 

In the Rails world, RESTful design and the CRUD interface are closely intertwined: the best illustration for this is that the resource generator generates a controller with all the CRUD actions in place (read is renamed to show, and delete is renamed to destroy). Also, there is the [DHH RailsConf '06 keynote](http://www.scribemedia.org/2006/07/09/dhh/) linking CRUD to RESTful design.

Why do we link those two concepts? Certainly not because this link was included in the original [Roy Fielding dissertation on the RESTful paradigm](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm). It is probably related to the fact that the CRUD actions match so nicely on the SQL statements in relational databases that most web applications are built on (SELECT, INSERT, UPDATE and DELETE) on the one hand, and on the HTTP methods that are used to access the web application on the other hand. So CRUD seems a logical link between the two.

But do the CRUD actions match nicely on the HTTP methods? DELETE is obvious, and the link between GET and read is also straightforward. Linking POST and create already takes a bit more imagination, but the link between PUT and update is not that clear at all. This is why PATCH was added to the HTTP spec and where the whole PUT/PATCH debate came from.

## Updates are not created equal

In the relational world of the database, UPDATE is just an operator that is part of set theory. In the world of publishing hypermedia resources that is HTTP, PUT is just a way to replace a resource on a given URL; PATCH was added later to patch up an existing resource in an application-specific way. 

But was it an update in the web application world? It turns out that it is not so clear cut. Most web application are built to support processes: it is an [OLTP system](http://en.wikipedia.org/wiki/Online_transaction_processing). A clear example of an OLTP system supporting a process is an e-commerce application. In an OLTP system, there is two kinds of data: master data of the objects that play a role within the context of your application (e.g. customer and product) and process-describing data, the *raison d'être* of your application (e.g., an order in the e-commerce example). 

For master data, the semantics of an update are clear: the customer has a new address, or a products description gets rewritten <a href="#restful-note-1">[1]</a>. For process-related data it is not so clear cut: the process isn't so much updated, the state of the process is changed due to an event: a **transaction**. An example would be the customer paying the order. 

In this case, a database UPDATE is used to make the data reflect the new reality due to this transaction. The usage of an UPDATE statement actually is an implementation detail, and you can easily do that otherwise. For instance, the event of paying for an order could just as well be stored as a new record INSERTed into the `order_payments` table. Even better would be to implement the process as a state machine, [two concepts that are closely linked](http://www.shopify.com/technology/3383012-why-developers-should-be-force-fed-state-machines), and to store the transactions so you can later analyze the process.

## Transactional design in a RESTful world

RESTful thinking for processes therefore causes more harm than it does good. The RESTful thinker may design both the payment of an order and the shipping of an order both as updates, using the HTTP PATCH method:

```
    PATCH /orders/42 # with { order: { paid: true  } }
    PATCH /orders/42 # with { order: { shipped: true } }
```

Isn't that a nice DRY design? Only one controller action is needed, just one code path to handle both cases! 

But should your application in the first place be true to RESTful design principles, or true to the principles of the process it supports? I think the latter, so giving the different transactions different URIs is better:

```
    POST /orders/42/pay
    POST /orders/42/ship
```

This is not only clearer, it also allows you to authorize and validate those transactions separately. Both transactions affect the data differently, and potentially the person that is allowed to administer the payment of the order may not be the same as the person shipping it.

## Some notes on implementation and security

When implementing a process, every possible transaction should have a corresponding method in the process model. This method can specify exactly what data is going to be updated, and can easily make sure that no other will be updated unintentionally. 

In turn, the controller should call this method on the model. Using `update_attributes` from your controller directly should be avoided: it is too easy to forget appropriate protection for mass-assignment, especially if  multiple transactions in the process update different fields of the model. This also sheds some light in the protecting from mass-assignment debate: protection is not so much part of the controller or the model, but should be part of the **transaction**.

Again, using a [state machine](https://github.com/pluginaweek/state_machine) to model the process makes following this following these principles almost a given, making your code more secure and bug free. Moreover, it easily allows you to keep full history of the process, making it completely auditable.

## Improving Rails

Finally, can we improve Rails to reflect these ideas and make it more secure? Here are my proposals:

- Do not generate an `update` action that relies on calling `update_attributes` when running the resource generator. This way it won't be there if it doesn't need to be reducing the possibility of a security problem. 
- Ship with a state machine implementation by default, and a generator for a state machine-backed process model. Be opinionated!

These changes would point Rails developers into the right direction when designing their application, resulting in better, more secure applications.


* * *

<a name="restful-note-1"></a>
*[1]* You may even want to model changes to master data as transactions, to make your system fully auditable and to make it easy to return to a previous value, e.g. to roll back a malicious update to the `ssh_key` field in the `users` table.

<small> A big thanks to Camilo Lopez, Jesse Storimer, John Duff and Aaron Olson for reading and commenting on drafts of this article. </small>

