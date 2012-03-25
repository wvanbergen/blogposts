# RESTful thinking considered harmful

It has been interesting and at times amusing to watch the last couple of intense debates in the Rails community. Of particular interest to me are the two topics that ended up on the Rails blog itself: [using the PATCH HTTP method for updates](http://weblog.rubyonrails.org/2012/2/25/edge-rails-patch-is-the-new-primary-http-method-for-updates/) and [protecting attribute mass-assignment in the controller vs. in the model](Strong parameters: Dealing with mass assignment in the controller instead of the model).

They are interesting because they our both about the update part of the CRUD model. PATCH deals with updates directly, and most problems with mass-assignment occur with updates, not with creation of resources. 

## REST and CRUD

In the Rails world, RESTful design and the CRUD interface are closely intertwined: the best illustration for this is that the resource generator generates a controller with all the CRUD actions in place (read is renamed to show, and delete is renamed to destroy). Also, there is the [DHH RailsConf '06 keynote](http://www.scribemedia.org/2006/07/09/dhh/) linking CRUD to RESTful design.

Why do we link those two concepts? Certainly not because this link was included in the original [Robert Fleming dissertation on the RESTful paradigm](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm). It is probably related to the fact that the CRUD methods match so nicely on the SQL statements in relational that most web applications are build on (SELECT, INSERT, UPDATE and DELETE) on the one hand, and on the HTTP methods that are used to access the web application on the other hand. So CRUD seems a logical link between the two.

But do the CRUD actions match nicely on the CRUD actions? DELETE is obvious, the link between GET and read is also straightforward. Linking POST and create already takes a bit more imagination, but the link between PUT and update is not that clear at all. This is why PATCH was added to the HTTP spec and where the whole PUT/PATCH debate came from.

## Updates are not created equal


