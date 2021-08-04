# Sinatra and Active Record CRUD

## Objectives

1. Understand how to implement the basic CRUD actions using Sinatra.
2. Understand which controller actions render which views in implementing CRUD in Sinatra.

## Introduction

By now you should be familiar with how to perform the basic CRUD (Create, Read, Update, Delete) actions using Active Record. Let's take a moment and review. For this example, we'll use the class name `Model` to stand in for whatever model your app is working with (`Article`, `Student`, `Song`, you name it).

- Create: `Model.create`
- Read: `Model.all`/`Model.find(id_number)`
- Update: `Model.update`
- Delete: `Model.destroy`

Now, let's take a look at how to build a Sinatra app that allows a user to
implement these actions through the interface of the web. The details of what
code gets written in each controller action are left slightly vague — you're
going to need to do some careful thinking for yourself when it comes to building
this later. The purpose of this reading is to illustrate the connections between
the various controller actions and views needed to implement CRUD.

## Connecting Controller Actions to Views for Implementing CRUD

### Create

The "create" part of CRUD is implemented in Sinatra by building a route, or controller action, to render a form for creating a new instance of your model.

- The `get '/models/new'` route is created as a block in your controller. In
  this block, we can render an `.erb` file that contains our form. In this case,
  we'll call it `new.erb`.
- That form sends a `POST` request to another controller action,
  `post '/models'`. It is here that you place the code that extracts the form
  data from the `params` and uses it to create a new instance of your model
  class, something along the lines of
  `Model.create(some_attribute: params[:some_attribute])`.

### Read

There are two ways in which we can read data. We may want to "read" or deliver
to our user, _all_ of the instances of a class, or a _specific_ instance of a
class.

- The `get '/models'` controller action handles requests for _all_ instances of
  a class. It should load up all of those instances and set them equal to an
  instance variable: `@models = Model.all`. Then, it renders the `index.erb`
  view page.
- The `index.erb` view page will use erb to render all of the instances stored
  in the `@models` instance variable.
- The `get '/models/:id'` controller action handles requests for a given
  instance of your model. For example, if a user types in
  `www.yourwebsite.com/models/2`, this route will catch that request and get the
  `id` number, in this case `2`, from the params. It will then find the instance
  of the model with that id number and set it equal to an instance variable:
  `@model = Model.find(params[:id])`. Finally, it will render the `show.erb`
  view page.
- The `show.erb` view page will use erb to render the `@model` object.

### Update

To implement the update action, we need a controller action that renders an
update form, and we need a controller action to catch the post request sent by
that form.

- The `get '/models/:id/edit'` controller action will render the `edit.erb` view
  page.
- The `edit.erb` view page will contain the form for editing a given instance of
  a model. This form will send a `PATCH` request to `patch '/models/:id'`.
- The `patch '/models/:id'` controller action will find the instance of the
  model to update, using the `id` from `params`, update and save that instance.

We'll need to update `config.ru` to use the Sinatra Middleware that lets our app
send `patch` requests.

`config.ru`:

```rb
use Rack::MethodOverride
run ApplicationController
```

From there, you'll need to add a line to your form.

`edit.erb`:

```html
<form action="/models/<%= @model.id %>" method="post">
  <input id="hidden" type="hidden" name="_method" value="patch" />
  <input type="text" ... />
</form>
```

The `MethodOverride` middleware will intercept every request sent and received
by our application. If it finds a request with `name="_method"`, it will set the
request type based on what is set in the `value` attribute, which in this case
is `patch`.

### Delete

The delete part of CRUD is a little tricky. It doesn't get its own view page but
instead is implemented via a "delete button" on the show page of a given
instance. This "delete button", however, isn't really a button; it's a form! The
form should send a `DELETE` request to `delete '/models/:id'` and should contain
only a "submit" button with a value of "delete". That way, it will appear as
only a button to the user. Here's an example:

```html
<form method="post" action="/models/<%= @model.id %>">
  <input id="hidden" type="hidden" name="_method" value="DELETE" />
  <input type="submit" value="delete" />
</form>
```

The hidden input field is important to note here. This is how you can submit
PATCH and DELETE requests via Sinatra. The form tag `method` attribute will be
set to `post`, but the hidden input field sets it to `DELETE`.

### Conclusion

Remember, the purpose of this reading is to help you understand which controller
actions render which views, and which views have forms that send requests to
which controller actions, as we implement CRUD. Check out the diagram below for
the big picture:

![](https://i.imgur.com/4o3Qtrv.png)
