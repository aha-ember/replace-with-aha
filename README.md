## replaceWith() vs transitionTo()

In this ember aha scenario the application we create will contain a parent route with some template code and an `{{outlet}}` for rendering child routes. The parent will simply rediect to a default child route.

### The application
This application will organize placed orders into different statuses.
Lets break this down by steps.

Our app should:
- [ ] contain the routes `/orders`, `/orders/new`, and `orders/shipped`
- [ ] navigate `/orders`, `/order/new` and `/orders/shipped` using links
- [ ] redirect to `/orders/new` when visiting `/orders`
- [ ] navigate using the browser history


Now that we have a clear idea of what needs to be done, lets begin.

### Create a new app and make some routes
Run in your terminal

`$ ember new replace-with-aha`

This will create a new ember application in the folder 'replace-with-aha' for us to work with. Lets change directories into our new app and generate some routes so we can check off the first (and maybe second?) step.

Inside our new ember application lets run our route generators.

`$ ember generate route orders`

`$ ember generate route orders/new`

`$ ember generate route orders/shipped`

Before we serve up our app lets add some text to our route templates.

Lets open up `app/templates/orders.hbs` and add some minimal code.
This is our main template file for our orders route so lets can add some links and makes sure we have our `{{outlet}}` for our child routes to render in.
```hbs
<h3>{{#link-to 'orders'}}Orders{{/link-to}}</h3>
<p>Filter By Status
{{#link-to 'orders.new'}}New{{/link-to}}
{{#link-to 'orders.shipped'}}Shipped{{/link-to}}
</p>
{{outlet}}
```

Next lets open up `app/templates/orders/new.hbs` and add some minimal html to indicate our new status.
```hbs
<p>New</p>
```

And then lets open up `app/templates/orders/shipped.hbs` and add some minimal html to indicate our shipped status.
```hbs
<p>Shipped</p>
```
Finally Lets serve our app and check our routes.

`$ ember serve`

Lets visit [http://localhost:4200/orders](http://localhost:4200/orders) and navigate through our new route. Looks like we can check of steps one and two.

Our app should:
- [x] contain the routes `/orders`, `/orders/new`, and `orders/shipped`
- [x] navigate `/orders`, `/order/new` and `/orders/shipped` using links
- [ ] redirect to `/orders/new` when visiting `/orders`
- [ ] navigate using the browser history

### The transitonTo replaceWith

If we look at the [emberjs guides for routes](https://guides.emberjs.com/v2.7.0/routing/redirection/) and check out the redirecting section we will see how easy it is to add redirects in ember. We can hook into the `beforeModel()` method hook an perform a transition. Ember has us covered! Let's do this.

Open up `app/routes/orders.js` and add the following code;
```javascript
import Ember from 'ember';

export default Ember.Route.extend({
  beforeModel(){
    this.transitionTo('orders.new');
  }
});
```

That's it. Let see if we can check off step 3. Visit the [http://localhost:4200/orders](http://localhost:4200/orders) url again and lets check for our redirect. Satisfied? If you are, try clicking on the orders link or hitting back on the browser. No redirect. No child template rendered in our `{{outlet}}`. We essentially created an accessible route that renders just template code and no valuable model data.

When ember entered the route `/orders` it fired the route method hooks and performed the `this.transitionTo('orders.new')` as expected. When we click on the orders link from a nested route such as `/orders/shipped` we never actually leave and re-enter the `/orders` route. The route methods hooks for `/orders` don't fire again.

### replaceWith() == redirect
Ember has a solution and it's quite simple. Lets take a look at the emberjs api for routes. You can find it here. [http://emberjs.com/api/classes/Ember.Route.html](http://emberjs.com/api/classes/Ember.Route.html)

Take a look at all those methods! In there you will find a method called `replaceWith()`. This is how `replaceWith()` is defined in the ember documentation.

*"Transition into another route while replacing the current URL, if possible. This will replace the current history entry instead of adding a new one. Beside that, it is identical to transitionTo in all other respects. See 'transitionTo' for additional information regarding multiple models."*

Perfect! We want our `/orders` route wiped from history when we do our redirect. Let change our code to reflect this change.

Open up `app/routes/orders.js` and change the `this.transitionTo('orders.new')` with `this.replaceWith('orders.new')`.
```javascript
import Ember from 'ember';

export default Ember.Route.extend({
  beforeModel(){
    this.replaceWith('orders.new');
  }
});
```

Visit the [http://localhost:4200/orders](http://localhost:4200/orders) again.  We will be redirected to `/orders/new`, now hit the browsers back button. We no longer get sent to our blank template page with an empty `{{outlet}}`.

But there is one more caveat. Our `/orders` link when accessed from one of it's child routes will not hook into it's route methods again. While the browser now behaves like we need it to we still need to update any links to our default route.

Lets open up `app/templates/orders.hbs` and update our default route. This is just an example. Ideally the the route to our orders page would be a part of our global application navigation. Conceptually they would operate the same way.
```hbs
<h3>{{#link-to 'orders.new'}}Orders{{/link-to}}</h3>
<p>Filter By Status
{{#link-to 'orders.new'}}New{{/link-to}}
{{#link-to 'orders.shipped'}}Shipped{{/link-to}}
</p>
{{outlet}}
```
Step 4 complete!

Our app should:
- [x] contain the routes `/orders`, `/orders/new`, and `orders/shipped`
- [x] navigate `/orders`, `/order/new` and `/orders/shipped` using links
- [x] redirect to `/orders/new` when visiting `/orders`
- [x] navigate using the browser history
