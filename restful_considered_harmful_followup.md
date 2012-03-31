# RESTful thinking considered harmful - followup

My previous post [RESTful thinking considered harmful](http://www.shopify.com/technology/5898287-restful-thinking-considered-harmful) caused quite a bit of discussion yesterday. Unfortunately, many people seem to have missed the point I was trying to make. This is likely my own fault for focusing too much on the implementation, instead of the thinking process of developers that I was actually trying to discuss. For this reason, I would like to clarify some points.

- My post was not intended as an arguments against REST. I don't claim to be a REST expert, and I don't really care about REST semantics.
- I am also not claiming that it is impossible to get the design right using REST principles in Rails.

So what was the point I was trying to make?

- Rails actively encourages the REST = CRUD design pattern, and all tutorials, screencasts, and documentation out there focuses on designing RESTful applications.
- However, REST requires developers to realize "publishing a blog post" is a resource, which is far from intuitive. This causes many new Rails developers to abuse the update action.
- Abusing update makes your application lose valuable data. This is irrevocable damage.
- Getting REST wrong may make your API less intuitive to use, but this can always be fixed in v2.
- Getting a working application that properly supports your process should be your end goal, having it adhere to REST principles is just a means to get there.
- All the focus on RESTful design and discussion about REST semantics makes new developers think this is actually more important and messes with them getting their priorities straight.

In the end, having a properly working application that doesn't lose data is more important than getting a proper RESTful API. Preferably, you want to have both, but you should always start with the former.

## Improving the status quo

In the end, what I want to achieve is educating developers, not changing the way Rails implements REST. Generators, screencasts and tutorials

- Rails could ship with a state machine implementation, and a generator to create a model based on it. Tutorials, screencasts, and documentation could focus on using it to design your application. This would lead to to better designed application with less bugs and security issues.
- You can always wrap your state machine in a RESTful API if you wish. But this should always come as step 2.

Hopefully this clarifies a bit better what I was trying to bring across.
