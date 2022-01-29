# Duck Fashion Show

## Submission Instructions

Welcome to WDB's backend project for development branch applicants — Spring 2022 👋

Make sure you read these instructions carefully before your start. If you have any questions
please reach out to [our email](webatberkeley@gmail.com).

To submit your project, please place your submission into a GitHub repo that is set to private. You
will be submitting your code on [Gradescope](https://www.gradescope.com/). If you do not have a
Gradescope account, please create one and if you are unable to create one, please email us
immediately. The Gradescope course code is `5VG683`. You will see two different assignments:
`Frontend Technical Project` and `Backend Technical Project`. _Please only submit to Backend
Technical Project._ You can ignore Frontend Technical Project.

The technical project will be due by ENTER DATE at midnight. We will be unable to respond to clarification emails sent in after then. so if you have any questions about the project, please let us know before then (we will be hosting technical project office hours in our club recruitment Discord, which you can join [here](https://discord.gg/syhqwpnx7a))

## Introduction

The organizers of the Duck Fashion Week have just called in, and they're in a bit of a predicament. They're aiming to host a fashion show for ducks across the country, but are having some difficulties because they didn't expect so many competitors to sign up. Duck Fashion Week has given us the important task of creating a system that will help them manage their clothing inventory as well their duck competitor information.

As a backend engineer, you need to create a backend service that helps keep track of individual ducks, the clothing that they have purchased & are wearing to the fashion week, and the overall results of the competition.

They've listed out a comprehensive set of rules for their event below (_make sure you read through this carefully_):

1. A duck can only REGISTER for the event if they have a valid name, and have a stipend associated with their name. You can access a list of all valid ducks and their stipends at THIS API. The response is formatted as {"duck name": stipend}. You may assume that no two ducks with the same name will ever register. You can find a more detailed explanation of what a duck is HERE.

2. Duck Fashion Week Administrators can ADD new clothing items or UPDATE existing clothing items at any time. You can find a more detailed explanation of what a clothing item is, as well as what updating one looks like, HERE.

3. Registered ducks will be able to execute a BID command for clothing items currently on the market. You can find a more detailed explanation of what a bid is HERE.

4. Whenever the Duck Fashion Week admins execute a SELL command, they clothing item will be sold. For this given clothing item, it will continue to be sold until there are either no more bids for the clothing item, or no more quantity available for the clothing item. You can find a more detailed explanation of what a sell is HERE.

   - As long as a clothing item can be sold, it will be sold to the bid with the highest price. After this, that bid will be deleted, the price of the bid will be deducted from the duck's stipend, and the clothing item is added to the duck's wardrobe. However, the transaction for a bid can only go through if the duck that issued the bid **has enough money in their stipend** to cover the price of the bid. If the duck does not have enough money, this bid should be ignored/deleted, and we should move on to finding the next best bid.

   - If the Duck Fashion Week admins execute a SELL for a clothing item but there are no more clothing items to sell, do nothing (and return with a status of 400).

5. At any point, the Duck Fashion Week Administrators can execute a TALLY command. When this happens, they want the following information:

   - For each duck registered in the competition, take the total sum of the Style Points associated with each purchased clothing item in its wardrobe. Return a dictionary/map of this information.

Here's a couple of example scenarios of how the event might run:

For the rest of this doc, assume the Valid Ducks API Endpoint response looks like this:

```
{
    "Alice": 100,
    "Bob": 60,
    "Clarence": 155,
    "Devon": 130
}
```

1. POST /register

```
{
    "name": "Alice"
}
```

Create a duck named "Alice" with a total stipend of 100.

2. POST /clothingItem

```
{
    "name": "Jacket",
    "units": 1,
    "points": 10
}
```

Creates a clothing item named "Jacket" with 1 unit worth 10 points.

3. POST /register

```
{
    "name": "Clarence"
}
```

Create a duck named "Clarence" with a total stipend of 155.

4. POST /clothingItem

```
{
    "name": "Pants",
    "units": 2,
    "points": 15
}
```

Creates a clothing item named "Pants" with 2 units, each worth 15 points.

5. POST /clothingItem

```
{
    "name": "Jacket",
    "units": 3,
    "points": 20
}
```

In this scenario, the clothing item named "Jacket" already existed. Thus we should UPDATE that instead of adding a completely new one. The new quantity should be 4 (comes from 1 + 3). The points should be averaged between all units (10 + 3 \* 20) / 4 = 17.5. Thus, "Jacket" now has 4 units and each is 17.5 points.

6. POST /bid

```
{
    "duck": "Alice"
    "name": "Jacket",
    "offer": 70,
}
```

Alice submits a bid for the jacket for a price of 70.

7. POST /bid

```
{
    "duck": "Alice"
    "name": "Pants",
    "offer": 50,
}
```

8. POST /bid

```
{
    "duck": "Clarence"
    "name": "Pants",
    "offer": 100,
}
```

9. POST /bid

```
{
    "duck": "Clarence"
    "name": "Pants",
    "offer": 55,
}
```

10. POST /sell

```
{
    "name": "Pants"
}
```

The bids, sorted in order of price are:

- bid of 100 from Clarence
- bid of 55 from Clarence
- bid of 50 from Alice

Bid of 100 from Clarence is processed (he has )

Here's another example of how to match a list of cats to a list of vegetable:

```
cats = ["Alex", "Abhi", "Samarth"]
veges = ["Artichoke", "Asparagus", "Onion", "Green Beans", "Squash"]

Matchings = {
  "Alex": ["Artichoke", "Asparagus"],
  "Abhi": ["Artichoke", "Asparagus"],
  "Samarth": ["Squash"],
}

```

## API Doc

To summarize, Duck Fashion Show is tasking us with building an API that can do the following things:

1. Add a valid duck to the database.
2. Add or update clothing items in the database.
3. Add a bid to the database.
4. Sell a clothing item in the database.
5. Tally the style points accumulated from all of the registered ducks in the database.

In the language of your choice, implement their API! Here are more technical details to help
you with your implementation:

### Register Valid Duck (`POST`)

The Duck Fashion Show already keeps a mapping of valid ducks names to initial stipends in their server. You can access them via sending a GET request to this API:

1. http://duck-fashion-show.herokuapp.com/api/ducks

When you receive a POST request to the `/register` endpoint, you should cross-reference the mapping of valid ducks. If the duck is invalid or something is wrong with the request. If the duck is valid, store its data in a database, and return the Response with status 200 and a body that contains the duck's info. If the duck is not valid, return an empty response with a error response (use your best judgment for which HTTP status codes to use).

```
ducks
POST /register Request Body { name: "Alice"}
POST /register Response Body (Status 200):
{
  name: "Alice",
  money: 100
}
```

### Get Current Duck Score (`GET`)

At any point, we should be able to get a duck's score.
Besides the list of cats & veges that they already have, the Cats in the Sky Company also wants to be able to add new cat / new vege easily. However, it turns out they don't have any space to store any data at all! We need to create a database for them and store all the cat / vege that they want to add.

This should be a POST endpoint that allow the users to add new cat / new vege by including the information in the post body.
For instance, if the user wants to add a cat called "Alice", they need to send a POST request with the following body:

```
{
  "cat": "Alice",
}
```

Similarly, if the user wants to add a vege, they need to send a POST request with a body containing `"vege": "name of vege"` as well.

Once the user sends a POST request, the data should be stored in a database. In the future, if the user call the "GET" endpoint again, we need to take the newly added data into account as well.

For instance:
if we have:

```
cats = ["Alex", "Abhi", "Samarth"]
veges = ["Artichoke", "Asparagus", "Onion", "Green Beans", "Squash"]
```

returned from the APIs, and the user decides to add a cat named "Oliver", then if the user send a GET request again, they should receive:

```
Matchings = {
  "Alex": ["Artichoke", "Asparagus"],
  "Abhi": ["Artichoke", "Asparagus"],
  "Samarth": ["Squash"],
  "Oliver": ["Onion"]
}
```

### Deleting a Vege (`DELETE`)

The Cats in the Sky Company also asks us to come up with a way to delete any vege in case any cat has some hidden alergies. To delete a vege, similarly to adding a cat / vege, the user needs to send a DELETE request, with body containing the name of the vege like this:

```
{
  "vege": ["Orange"]
}
```

There are two different cases for DELETE:

1. If the vege is one of the veges returned by the vege API, track it inside the database, and make sure that when calling `GET`, that vege will not be considered during the matching process.
2. If the vege is not one of the veges returned by the vege API, simply remove it from the database.

Here's a example of the first case:

```
# Returned by API
cats = ["Alex", "Abhi", "Samarth"]
veges = ["Artichoke", "Asparagus", "Green Beans", "Squash"]

# First Get
Matchings = {
  "Alex": ["Artichoke", "Asparagus"],
  "Abhi": ["Artichoke", "Asparagus"],
  "Samarth": ["Squash"],
}

# Send the DELETE request
DELETE: {
  "vege": "Squash"
}

# GET after DELETE
Matchings = {
  "Alex": ["Artichoke", "Asparagus"],
  "Abhi": ["Artichoke", "Asparagus"],
}
```

## Optional: Authentication

_This is an extra credit question. Please prioritize solving the other questions before attempting this question._

Some evil customers are trying to delete our cats by sending DELETE request to our server! Stop them from doing that by building a user authentication system. The system should support the following functionality:

1. Sign Up: the user should be able to sign up via a POST request, containing the {"username": "name","pwd": "password"} request body.
2. Sign In: the user should be able to sign in via a POST request, containing the {"username": "name","pwd": "password"} request body.
3. The DELETE route can only be accessed if the user is signed in.

_Hint: Access Token_

## Assumptions

There are many details that are left intentionally vague. Though you are very much welcome to
email us to ask for clarifications, we will most likely tell you to use your best judgement.
Because of this, feel free to create a `assumptions.md`, where you can type out and
voice any assumptions you made throughout this project. We also _highly_ encourage you to
write out your own documentation to this API and provide us a glimpse of your rationale
behind every design decision.
