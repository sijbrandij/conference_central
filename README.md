App Engine application for the Udacity training course.

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.


[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool

## Design Explanations
1. Task 1: Sessions - sessions are a data model with a link to the conference (through assigning Conference as the ancestor of Session and storing the conferenceId) they are a part of. I chose StringProperty for name, highlights, speaker and typeOfSession because I feel the StringProperty has sufficient options to work with for those properties. I chose IntegerProperty for conferenceId, Duration and wishlistCount because I feel these properties would be properties one would want to do calculations with. I chose a DateProperty for date and a TimeProperty for startTime because I feel these best represent the properties' functions. For speaker, I chose not to create a full-blown model because the current featureset does not require the speaker to be a data-model. I like to keep things like data structure etc. simple if that's possible. When the app would require more functionality for speakers, for instance speaker management by the conference organization, then I would turn speaker into a data model.
1. Task 2: Wishlists - chose to create sessions as a table with the userId and one or multiple sessionIds. This way, it is easy to find a user's wishlist and the sessions they wishlisted. I believe this is an efficient way of storing this data.
1. Indexes and queries - one query I thought of is the getWishlistBySession query: find the users who wishlisted a particular session. The organizer can use this query to drive more users to their conferences. The other query I came up with is mostPopular: a query to determine which session is most popular among users. I added indexes for the Session queries BySpeaker and ByType, and I added an index for Wishlist, both for userId and sessionKey. My comments about the query problem are below.
1. Tasks - implemented using the Task Queue, Memcache key is MEMCACHE_SPEAKERS_KEY

## Query problem (task 3)
The problem with the query to search for all non-workshop sessions before 7 pm is that Datastore does not allow for more than one inequality query on multiple fields in one query. In this case, there are two inequality queries for two separate fields:
1. TypeOfSession != 'workshop'
1. startTime < 19:00:00

The solution to this is to change the queries to a query with only one inequality. Based on the data I changed the query into:
1. query1 = Session.query(ancestor=conference.key)
1. query2 = query1.filter(Session.typeOfSession.IN(['lecture', 'keynote'])) # filter for non-workshop sessions
1. query3 = query2.filter(Session.startTime < time) # filter for sessions starting before a time object
