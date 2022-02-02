# Duck Fashion Show

## Clarifications
- Your database can definitely be on localhost! We don't expect you to pay for hosting it in the cloud.
- Remember that you don't need to finish the full project to get a full score! Though it would be great if you are able to, we will not hold it against you if you cannot.
- Some changes were made to the examples in this doc. The ground source of truth is the API documentation on stoplight, but since we made changes late, we will accept either:
    - changed name to clothingItem for SELL and BID
    - changed /duck/register to /duck
## Submission Instructions

Welcome to WDB's backend project for development branch applicants — Spring 2022 👋

Make sure you read this ENTIRE DOCUMENT, especially these instructions, carefully before you start. If you have any questions
please reach out to [our email](webatberkeley@gmail.com).

To submit your project, please place your submission into a GitHub repo that is set to private. You
will be submitting your code on [Gradescope](https://www.gradescope.com/). If you do not have a
Gradescope account, please create one and if you are unable to create one, please email us
immediately. The Gradescope course code is `5VG683`. You will see two different assignments:
`Frontend Technical Project` and `Backend Technical Project`. _Please only submit to Backend
Technical Project._ You can ignore Frontend Technical Project.

The technical project will be due by Wednesday, 2/2 at midnight. We will be unable to respond to clarification emails sent in after then. so if you have any questions about the project, please let us know before then (we will be hosting technical project office hours in our club recruitment Discord, which you can join [here](https://linktr.ee/webdevatberkeley)).

Also, this page may potentially keep changing if we get some frequently asked questions, so keep this repository bookmarked and check back on it every now and then! If there are major changes however, we'll make sure to email you about those.

## Introduction

The organizers of the Duck Fashion Week have just called in, and they're in a bit of a predicament. They're aiming to host a fashion show for ducks across the country, but are having some difficulties because they didn't expect so many competitors to sign up. Duck Fashion Week has given us the important task of creating a system that will help them manage their clothing inventory as well their duck competitor information.

As a backend engineer, you need to create a backend service that helps keep track of individual ducks, the clothing that they have purchased & are wearing to the fashion week, and the overall results of the competition.

They've listed out a comprehensive set of rules for their event below (_make sure you read through this carefully_):

1. A duck can REGISTER for the event if they provide their name and initial stipend. You may assume that no two ducks with the same name will ever register. You can find a more detailed explanation of what a duck is in the API Documentation section.

2. Duck Fashion Week Administrators can ADD new clothing items or UPDATE existing clothing items at any time. You can find a more detailed explanation of what a clothing item is in the API Documentation section, but as a quick rundown, its got a "name", a "units" (or quantity), and "points" (the average # of points that each unit is worth).

3. Registered ducks will be able to execute a BID command for clothing items currently on the market. You can find a more detailed explanation of what a bid is in the API Documentation section, but as a quick rundown, its got a "duck" (name of duck buying), an offer (price), and the clothingItem (name of item)

4. Whenever the Duck Fashion Week admins execute a SELL command, they clothing item will be sold. For this given clothing item, it will continue to be sold until there are either no more bids for the clothing item, or no more quantity available for the clothing item. You can find a more detailed explanation of what a sell is in the API Documentaiton section.

   - As long as a clothing item can be sold, it will be sold to the bid with the highest price. After this, that bid will be deleted, the price of the bid will be deducted from the duck's stipend, and the clothing item is added to the duck's wardrobe. However, the transaction for a bid can only go through if the duck that issued the bid **has enough money in their stipend** to cover the price of the bid. If the duck does not have enough money, this bid should be ignored/deleted, and we should move on to finding the next best bid.

5. At any point, the Duck Fashion Week Administrators can execute a TALLY command. When this happens, they want the following information:

   - For each duck registered in the competition, take the total sum of the points associated with each purchased clothing item in its wardrobe. Return a dictionary/map of this information. Note: if the points have changed for an item since they have bought it (maybe an POST /clothingItem happened after the /POST sell), use the most recent points value!

Here's a couple of example scenarios of how the event might run (we potentially might add more, in case there is popular demand):

1. POST /duck

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

3. POST /duck

```
{
    "name": "Clarence",
    "money": 155
}
```

4. GET /duck

```
{
    "name": "Clarence",
    "money": 155,
    "clothingItems": []
}
```

Return info on Clarence. Note that since he has no clothing items yet, we return an empty array.

5. POST /clothingItem

```
{
    "name": "Pants",
    "units": 2,
    "points": 15
}
```

Creates a clothing item named "Pants" with 2 units, each worth 15 points.

6. POST /clothingItem

```
{
    "name": "Jacket",
    "units": 3,
    "points": 20
}
```

In this scenario, the clothing item named "Jacket" already existed. Thus we should UPDATE that instead of adding a completely new one. The new quantity should be 4 (comes from 1 + 3). The points should be averaged between all units (10 + 3 \* 20) / 4 = 17.5. Thus, "Jacket" now has 4 units and each is 17.5 points.

7. POST /transact/bid

```
{
    "duck": "Alice"
    "clothingItem": "Jacket",
    "offer": 70,
}
```

Alice submits a bid for the jacket for a price of 70.

8. POST /transact/bid

```
{
    "duck": "Alice"
    "clothingItem": "Pants",
    "offer": 50,
}
```

9. POST /transact/bid

```
{
    "duck": "Clarence"
    "clothingItem": "Pants",
    "offer": 100,
}
```

10. POST /transact/bid

```
{
    "duck": "Clarence"
    "clothingItem": "Pants",
    "offer": 55,
}
```

11. POST /transact/bid

```
{
    "duck": "Clarence"
    "clothingItem": "Jacket",
    "offer": 20,
}
```

12. POST /transact/sell

```
{
    "clothingItem": "Pants"
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
    "clothingItem": "Jacket"
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

## API Requirements: what you're working on

To summarize, Duck Fashion Show is tasking us with building an API that can do the following things:

1. Add a duck to the database.
2. Retrieve info on a duck in the database.
3. Add or update clothing items in the database.
4. Add a bid to the database.
5. Sell a clothing item in the database.
6. Tally the points accumulated from all of the registered ducks in the database.

Now it's up to you to implement their API in the language of your choice! You can find a detailed API doc at https://wdb.stoplight.io/docs/backend-technical-project/b3A6MzgxMDMwOTQ-register-duck.

Some of these routes don't need much logic (ex: 1, 2, and 4), while others will require a bit more thinking (3, 5, 6). If you aren't able to finish all of the routes for the project, **don't worry**! It's supposed to be challenging, and we don't expect everyone to finish it, especially folks who are interested in joining the bootcamp. You won't need to complete the full project in order to "pass"/get a full score :)

Note: we **highly** recommend using MongoDB as your database for this project, although if you don't have experience with it, any NoSQL database is also a great alternative. If none of those work however, you are still welcome to use other alternatives.

It is also encouraged to use JavaScript/TypeScript with Node.js for your backend, but it is completely fine if you would rather use a different stack.

## Design Doc

In addition to building out this API, you will need to write up a short design doc (designdoc.md). We don't intend for this to take very much time, but we want to hear some of the choices you made and why. To be specific, here are some points you might want to talk about:

- why did you choose to organize your data in this particular way?
- can talk a bit about the "harder" routes that you worked on — harder is completely subjective, so feel free to get creative here!
- how did you decide on certain response codes?

This should be at most a page, so feel free to be brief!

## Optional: Authentication

_This is an extra credit question. Please prioritize solving the other questions before attempting this question._

Some evil ducks are trying to spoof fake bids from other ducks. Stop them from doing that by building a duck authentication system. The system should support the following functionality:

1. Register: the duck should still be able to register via a POST request. However, now you should include a password field in the post request.
2. Login: this is a completely new route you will need to create. The duck should be able to login via a POST request, containing the {"name": "duck_name_goes_here", "password": "password_goes_here"} request body.
3. The /transact/bid route can only be accessed if the duck is signed in. Furthermore, an authenticated duck can only make a bid in their own name.

_Hint: Access Token_

## Assumptions

There are many details that are left intentionally vague. Though you are very much welcome to
email us to ask for clarifications, we will most likely tell you to use your best judgement.
Because of this, feel free to create a `assumptions.md`, where you can type out and
voice any assumptions you made throughout this project. We also _highly_ encourage you to
write out your own documentation to this API and provide us a glimpse of your rationale
behind every design decision.
