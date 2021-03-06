# API Basics

## Creating a Card

In order to access any of the Cloud App APIs, you first must create a `Card` instance inside of your App.  The `SW` global namespace will be used for all Spiceworks-related APIs, including the `Card` constructor:

```js
var card = new SW.Card();
```

Once you've created a Card object, you can use the card to make requests and respond to events from Spiceworks.

## Card Lifecycle

When an App Card is first loaded inside of Spiceworks the `activate` event will be fired.  Your App can respond to the `activate` event by passing a callback to the `onActivate` method:

```js
var card = new SW.card();
card.onActivate(function(envData){
  // do any app setup or configuration
});
```

Your callback function will receive a single argument that is an object containing data about the environment in which your Card was loaded.

## Card Services

The Canvas App APIs are divided into groups called `services`.  Each service provides its own scope within Spiceworks.  For example, the `helpdesk` service provides your app access to read and write  user data related to the Spiceworks Helpdesk.  When a Spiceworks user downloads your app, they can decide whether to give your app access to each service.

To access a service, simply call the `Card#services` method and pass in the name of the service you want, e.g.:

```js
var card = new SW.Card();
card.services('helpdesk');
```

The `services` method will then return a `CardService` object that responds to requests.  For a full list of the available services see [Canvas App Services](https://github.com/spiceworks/spiceworks-js-sdk/blob/master/docs/apis/helpdesk.md).

## Requests

Each service will respond to a specific set of API requests.  To make a request to a service, you simply call the `CardService#request` method.  For example if you wanted to get the list of tickets for the current user, you simply request `tickets` from the `helpdesk` service:

```js
var card = new SW.Card();
card.services('helpdesk').request('tickets').then(function(data){
  console.log(data);
});

/* prints to the console:
 * {
 *   meta: {...},
 *   tickets: [...]
 * }
 *
 */
```

Requests will always return a response from the service.  Services will respond asynchronously with a promise object that responds to a method `then`, as defined by the [Promises/A+](http://promises-aplus.github.com/promises-spec/) specification.

### Parameters

Requests can also handle arguments that help fulfill a request.  For instance, to get a single ticket from the `helpdesk` service, you must include a ticket `id` with the `ticket` request:

```js
var card = new SW.Card();
card.services('helpdesk').request('ticket', 2)
  .then(function(ticket){
    // do something with the ticket...
  });
```

All requests accept a final, optional parameter that specifies options to the request method.  For example, the `tickets` request allows you to optionally filter the list of tickets returned.  So, if you wanted just the list of currently open tickets, you could write the following:

```js
var card = new SW.Card();
card.services('helpdesk').request('tickets', { status: 'open' })
  .then(function(data){
    // do something with tickets...
  });
```

For a full list of the supported requests for a service visit the [service documentation page](https://github.com/spiceworks/spiceworks-js-sdk/blob/master/docs/apis/helpdesk.md).

### Paging

Requests that return multiple items will be paginated.  The pagination will keep from returning too many items at once, which can slow down your app or the Spiceworks server.  

For paginated requests, you can request a specific page of results.  For some paginated requests, you can also specify the number of results per page.  To set these parameters pass them to the `request` function as keys in the final options argument.

For example, if you wanted the second page of currently open tickets with 50 tickets per page you would write:

```js
var card = new SW.Card();
card.services('helpdesk').request('tickets', { status: 'open', page: 2, per_page: 50 })
```

If you do not pass in a `page` value, then a request will always return the first page (page `1`) of results.  Different resources will have a different default `per_page` value and may have different limits.  See the specific request documentation for more information on the `per_page` defaults.

When a paginated request is returned, it will always have a top-level `meta` attribute containing pagination information.  For example:

```js
"meta": {
  "total_entries": 205, // total number of items, across all pages
  "page_count": 7, // total number of pages
  "per_page": 30, // number of items per page
  "current_page": 2 // the current page number
}
```

## Error Handling

Your app should gracefully handle errors returned by a request.  Since all requests return a promise object, this simply means providing an error handler to the `then` function:

```js
var card = new SW.Card();
card.services('helpdesk').request('tickets')
   .then(undefined, function(errors){
    // handle errors
   });
```

When an error occurs on a request, the promise will be rejected with an object containing a list of error objects.  For instance, in the previous example if your app did not have access to the `helpdesk` service, the promise would be rejected with the following object:

```js
{
  errors: [{
    title: 'Connection Error',
    details: 'Could not connect to service helpdesk. Make sure the service name is correct and that your App has access to this service.'
  }]
}
```
