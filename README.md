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

1. A duck can only REGISTER for the event if they have a valid name, and have a stipend associated with their name. You can access a list of all valid ducks and their stipends at THIS API. The response is formatted as {"duck name": stipend}. You may assume that no two ducks with the same name will ever register. You can find a more detailed explanation of what a duck is in the API Documentation section.

2. Duck Fashion Week Administrators can ADD new clothing items or UPDATE existing clothing items at any time. You can find a more detailed explanation of what a clothing item is, as well as what updating one looks like in the API Documentation section.

3. Registered ducks will be able to execute a BID command for clothing items currently on the market. You can find a more detailed explanation of what a bid is in the API Documentation section.

4. Whenever the Duck Fashion Week admins execute a SELL command, they clothing item will be sold. For this given clothing item, it will continue to be sold until there are either no more bids for the clothing item, or no more quantity available for the clothing item. You can find a more detailed explanation of what a sell is in the API Documentaiton section.

   - As long as a clothing item can be sold, it will be sold to the bid with the highest price. After this, that bid will be deleted, the price of the bid will be deducted from the duck's stipend, and the clothing item is added to the duck's wardrobe. However, the transaction for a bid can only go through if the duck that issued the bid **has enough money in their stipend** to cover the price of the bid. If the duck does not have enough money, this bid should be ignored/deleted, and we should move on to finding the next best bid.

   - If the Duck Fashion Week admins execute a SELL for a clothing item but there are no more clothing items to sell, do nothing (and return with a status of 400).

5. At any point, the Duck Fashion Week Administrators can execute a TALLY command. When this happens, they want the following information:

   - For each duck registered in the competition, take the total sum of the points associated with each purchased clothing item in its wardrobe. Return a dictionary/map of this information. Note: if the points have changed for an item since they have bought it (maybe an POST /clothingItem happened after the /POST sell), use the most recent points value!

Here's a couple of example scenarios of how the event might run (we potentially might add more, in case there is popular demand):

1. POST /duck/register

```
{
    "name": "Alice",
    "money": 100,
}
```

Create a duck named "Alice" with a total stipend of 100.

2. POST /clothingItem

```
{
    "name": "Jacket",
    "units": 1,
    "points": 10,
}
```

Creates a clothing item named "Jacket" with 1 unit worth 10 points.

3. POST /duck/register

```
{
    "name": "Clarence",
    "money": 155
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

6. GET /clothingItem

```
Request:
{
    "name": "Jacket"
}


Response:
{
    "name": "Jacket",
    "units": 4,
    "points": 17.5
}
```

7. POST /transact/bid

```
{
    "duck": "Alice"
    "name": "Jacket",
    "offer": 70,
}
```

Alice submits a bid for the jacket for a price of 70.

8. POST /transact/bid

```
{
    "duck": "Alice"
    "name": "Pants",
    "offer": 50,
}
```

9. POST /transact/bid

```
{
    "duck": "Clarence"
    "name": "Pants",
    "offer": 100,
}
```

10. POST /transact/bid

```
{
    "duck": "Clarence"
    "name": "Pants",
    "offer": 55,
}
```

11. POST /transact/bid

```
{
    "duck": "Clarence"
    "name": "Jacket",
    "offer": 20,
}
```

12. POST /transact/sell

```
{
    "name": "Pants"
}
```

The bids, sorted in order of price are:

- bid of 100 from Clarence
- bid of 55 from Clarence
- bid of 50 from Alice

First, the bid of 100 from Clarence is processed. He now has a `money` of 55, but has Pants too. Then, the bid of 55 from Clarence is processed. He now has a `money` of 0, but has another pair of pants. There are no more units for Pants, so we are done.

13. GET /duck

```
Request Params (not body):
{
    "name": "Clarence"
}


Response:
{
    "name": "Clarence",
    "money": 0,
    "clothingItems": ["Pants", "Pants"]
}
```

14. POST /transact/sell

```
{
    "name": "Jacket"
}
```

The bids, sorted in order of price are:

- bid of 70 from Alice
- bid of 20 from Clarence

First, the bid of 70 from Alice is processed. She now has a `money` of 30, but has Jacket too. Then, the bid of 20 from Clarence is processed, but he has 0 `money` so he can't afford it, so the bid is deleted and he doesn't get anything. There are no more bids for Jacket, so we are done.

15. GET /duck

```
Request Params (not body):
{
    "name": "Alice"
}


Response:
{
    "name": "Alice",
    "money": 30,
    "clothingItems": ["Jacket"]
}
```

16. GET /tally

```
No Request Params


Response:
{
    "Alice": 17.5,
    "Clarence": 30
}
```

## API Doc

To summarize, Duck Fashion Show is tasking us with building an API that can do the following things:

1. Add a duck to the database.
2. Retrieve info on a duck in the database.
3. Add or update clothing items in the database.
4. Retrieve info on a clothing item in the database.
5. Add a bid to the database.
6. Sell a clothing item in the database.
7. Tally the points accumulated from all of the registered ducks in the database.
8. Clear Database

Now it's up to you to implement their API in the language of your choice! You can find a detailed API doc at https://wdb.stoplight.io/studio/backend-technical-project.

Note: we highly recommend using MongoDB as your database for this project, although if you don't have experience with it, any NoSQL database is also a great alternative. If none of those work however, you are still welcome to use other alternatives.

## Design Doc

In addition to building out this API, you will need to write up a short design doc (designdoc.md). We don't intend for this to take very much time, but we want to hear some of the choices you made and why. To be specific, here are some points you might want to talk about:

- why did you choose to organize your data schemas in this particular way?
- why did you organize your project (think file structure, organizing routes, etc) in this particular way?
- can talk a bit about the "harder" routes that you worked on — harder is completely subjective, so feel free to get creative here!
- how did you decide on certain response codes?

This shouldn't be longer than a page, so feel free to be brief!

## Optional: Authentication

_This is an extra credit question. Please prioritize solving the other questions before attempting this question._

Some evil ducks are trying to spoof fake bids from other ducks. Stop them from doing that by building a duck authentication system. The system should support the following functionality:

1. Register: the duck should still be able to register via a POST request. However, now you should include a password field in the post request.
2. Login: this is a completely new route you will need to create. The duck should be able to login via a POST request, containing the {"name": "duck_name_goes_here", "password": "password_goes_here"} request body.
3. The /bid route can only be accessed if the duck is signed in. Furthermore, an authenticated duck can only make a bid in their own name.

_Hint: Access Token_

## Assumptions

There are many details that are left intentionally vague. Though you are very much welcome to
email us to ask for clarifications, we will most likely tell you to use your best judgement.
Because of this, feel free to create a `assumptions.md`, where you can type out and
voice any assumptions you made throughout this project. We also _highly_ encourage you to
write out your own documentation to this API and provide us a glimpse of your rationale
behind every design decision.
