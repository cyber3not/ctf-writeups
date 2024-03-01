# Diogenes' Rage

## Description
> Having missed the flight as you walk down the street, a wild vending machine appears in your way. You check your pocket and there it is, yet another half torn voucher coupon to feed to the consumerism. You start wondering why should you buy things that you don't like with the money you don't have for the people you don't like. You're Jack's raging bile duct.

<br>
<br>

## Foreword

This challenge was  very cool and I was excited because it was my first experience with a __Race Condition__ and I learned a lot of new things.

Therefore, I would like to try to make this writeup a bit more detailed to be better prepared for the next vulnerability of this kind.

<br>
<br>

## Walkthrough

Well, in this challenge we were given an online instance and the challenge code.

There was not much to interact with the online instance itself in the browser to solve the challenge.

It was more necessary to look at the challenge code and understand what the application under the hood does and what functions it offers that we can't normally interact with in the browser.

But let's take a look at the web instance in the browser at the beginning anyway.

![Screenshot0](./screenshots/0.png)

We see a vending machine as you know it from the streets, where you have to throw money in to buy a snack or drink and at the end the machine hangs and you have to shake it for what you paid for....who doesn't know it!

We have 9 products that we can buy from this vending machine with different prices.

Among them is the product with the code __C8__ with an unusual price of `13.37`.

But more about this later.

At the bottom right we see a coupon code with the name `HTB_100` and the value of one dollar.

Now the interaction in the browser with the vending machine does not really work, so I looked at the code of the vending machine.

<br>

__Challenge files in Tree-Like format:__


```
├── Dockerfile
├── build-docker.sh
├── challenge
│   ├── database.js
│   ├── flag
│   ├── helpers
│   │   └── JWTHelper.js
│   ├── index.js
│   ├── middleware
│   │   └── AuthMiddleware.js
│   ├── package.json
│   ├── routes
│   │   └── index.js
│   ├── static
│   │   ├── css
│   │   │   ├── bootstrap.min.css
│   │   │   └── main.css
│   │   ├── images
│   │   │   ├── bat.gif
│   │   │   ├── brickwall.png
│   │   │   ├── clam-chowder.png
│   │   │   ├── coupon.png
│   │   │   ├── favicon.png
│   │   │   ├── film-reel.png
│   │   │   ├── ikea-catalog.png
│   │   │   ├── marlboro.png
│   │   │   ├── meow.gif
│   │   │   ├── motor-oil.png
│   │   │   ├── pavement.png
│   │   │   ├── rayhan.gif
│   │   │   ├── road.png
│   │   │   ├── sky.jpg
│   │   │   ├── soap.png
│   │   │   ├── starbucks.png
│   │   │   ├── vet-medicine.png
│   │   │   └── vinegar.png
│   │   └── js
│   │       ├── jquery-3.6.0.min.js
│   │       ├── jquery-ui.js
│   │       ├── jquery.marquee.min.js
│   │       └── main.js
│   └── views
│       └── index.html
└── config
    └── supervisord.conf
```

The first thing I always want to know is where the flag is located to get a first idea of what kind of vulnerability we might be dealing with.

As we can see from the `tree` command, it is under the challenge folder.

And as an `egrep` command showed me, it is read by javascript at a certain point.

![Screenshot1](./screenshots/1.png)

Okay, let's take a closer look at the __routes/index.js__ file, the code of which I will now insert here in full.

```js
const fs             = require('fs');
const express        = require('express');
const router         = express.Router();
const JWTHelper      = require('../helpers/JWTHelper');
const AuthMiddleware = require('../middleware/AuthMiddleware');

let db;

const response = data => ({ message: data });

router.get('/', (req, res) => {
	return res.render('index.html');
});

router.post('/api/purchase', AuthMiddleware, async (req, res) => {
	return db.getUser(req.data.username)
		.then(async user => {
			if (user === undefined) {
				await db.registerUser(req.data.username);
				user = { username: req.data.username, balance: 0.00, coupons: '' };
			}
			const { item } = req.body;
			if (item) {
				return db.getProduct(item)
					.then(product => {
						if (product == undefined) return res.send(response("Invalid item code supplied!"));
						if (product.price <= user.balance) {
							newBalance = parseFloat(user.balance - product.price).toFixed(2);
							return db.setBalance(req.data.username, newBalance)
								.then(() => {
									if (product.item_name == 'C8') return res.json({
										flag: fs.readFileSync('/app/flag').toString(),
										message: `Thank you for your order! $${newBalance} coupon credits left!`
									})
									res.send(response(`Thank you for your order! $${newBalance} coupon credits left!`))
								});
						}
						return res.status(403).send(response("Insufficient balance!"));

					})
			}
			return res.status(401).send(response('Missing required parameters!'));
		});
});

router.post('/api/coupons/apply', AuthMiddleware, async (req, res) => {
	return db.getUser(req.data.username)
		.then(async user => {
			if (user === undefined) {
				await db.registerUser(req.data.username);
				user = { username: req.data.username, balance: 0.00, coupons: '' };
			}
			const { coupon_code } = req.body;
			if (coupon_code) {
				if (user.coupons.includes(coupon_code)) {
					return res.status(401).send(response("This coupon is already redeemed!"));
				}
				return db.getCouponValue(coupon_code)
					.then(coupon => {
						if (coupon) {
							return db.addBalance(user.username, coupon.value)
								.then(() => {
									db.setCoupon(user.username, coupon_code)
										.then(() => res.send(response(`$${coupon.value} coupon redeemed successfully! Please select an item for order.`)))
								})
								.catch(() => res.send(response("Failed to redeem the coupon!")));
						}
						res.send(response("No such coupon exists!"));
					})
			}
			return res.status(401).send(response("Missing required parameters!"));
		});
});

router.get('/api/reset', async (req, res) => {
	res.clearCookie('session');
	res.send(response("Insert coins below!"));
});

module.exports = database => {
	db = database;
	return router;
};
```

So our flag is read in under this condition:

```js
<SNIP>

if (product.item_name == 'C8') return res.json({
	flag: fs.readFileSync('/app/flag').toString(),
	message: `Thank you for your order! $${newBalance} coupon credits left!`

<SNIP>
```

So when we purchase the product with the value __C8__, we get our flag!

Ok, that's good to know for a start, but how do we continue interacting with the application?

I made a note of all the API routes and created functioning curl requests to better understand the application.

We have two API endnodes that we can work with.

```
/api/purchase
/api/coupons/apply
```

Let's play around with them using curl and see what happens.

<br>

__Purchase:__ `curl -v -X POST http://94.237.49.138:48479/api/purchase`

![Screenshot2](./screenshots/2.png)

<br>

__Apply:__ `curl -v -X POST http://94.237.49.138:48479/api/coupons/apply`

![Screenshot3](./screenshots/3.png)

<br>

We get an HTTP status code `401 Unauthorized` for both and furthermore the application complains that parameters are missing: `{"message": "Missing required parameters!"}. `.

The application sets us a __JWT__ in the response, which by the way is always a new user.

Let's decode a few tokens:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyXzZjMDE0YTc4MDciLCJpYXQiOjE3MDkzMTEwNzd9.gdQiQQeoJFrwkVu5oMz2Tn8PomAJBVoKAd07YmqiHPE
{"alg":"HS256","typ":"JWT"}{"username":"tyler_6c014a7807","iat":1709311077}.<GIBBERISH_FROM_SIGNATURE>

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyXzA0OTE3MzcxMDEiLCJpYXQiOjE3MDkzMTIyMTN9.lwr7HXtWcoyBwoYkCzFlPl4gYKyE3TWY8TIsGv7snbw
{"alg":"HS256","typ":"JWT"}{"username":"tyler_0491737101","iat":1709312213}.<GIBBERISH_FROM_SIGNATURE>

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyX2M1YmI0YjM0NTQiLCJpYXQiOjE3MDkzMTIyNDR9.87L-paXBd0jjD40YgI0jUmaCOoEx5KuGfclXWr7C6Vc
{"alg":"HS256","typ":"JWT"}{"username":"tyler_c5bb4b3454","iat":1709312244}.<GIBBERISH_FROM_SIGNATURE>
```

So the application always gives us a new __JWT__ with a new username starting with `tyler_` followed by a few random bytes.

Here is a short code insight which generates the username from the file __AuthMiddleware.js__.

![Screenshot4](./screenshots/4.png)

I briefly thought that we might be dealing with a JWT hack and tried some things like setting the `alg` value to `none` and removing the Signature and so on, but that was just a quick rabbit hole.

There is also no weak JWT signing if we look at how the token is signed.

![Screenshot5](./screenshots/5.png)

Okay, now I crafted working curl requests to be able to deal with the application as intended.

I looked at the functions of the endnodes, which parameters they require and took one of the _JWT_'s for it and set it to be authorized by the application.

<br>

### Apply
> Here is a request how we load a user with the __HTB_100__ coupon worth __1$__.

__Command:__

```bash
curl -v -X POST http://94.237.49.138:48479/api/coupons/apply -H "Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyX2M1YmI0YjM0NTQiLCJpYXQiOjE3MDkzMTIyNDR9.87L-paXBd0jjD40YgI0jUmaCOoEx5KuGfclXWr7C6Vc" -H "Content-Type: application/json" --data '{"username":"tyler_c5bb4b3454","coupon_code":"HTB_100"}'
```

__Response:__

```json
{"message":"$1 coupon redeemed successfully! Please select an item for order."}
```

<br>

### Purchase
> Here a request how we buy the product __A1__ with a price of __0.55$__

__Command:__

```bash
curl -v -X POST http://94.237.49.138:48479/api/purchase -H "Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyX2M1YmI0YjM0NTQiLCJpYXQiOjE3MDkzMTIyNDR9.87L-paXBd0jjD40YgI0jUmaCOoEx5KuGfclXWr7C6Vc" -H "Content-Type: application/json" --data '{"username":"tyler_c5bb4b3454","item":"A1"}'
```

__Response:__

```json
{"message":"Thank you for your order! $0.45 coupon credits left!"}
```

Okay fine, we've already got to the point where we can legitimately interact with the vending machine, load coupons and buy an item.

But now we want to buy the item __C8__, which the flag promises us.

Unfortunately, the item is not cheap and we don't have enough credit for it.

![Screenshot6](./screenshots/6.png)

I could not recharge the code twice (...not yet!).

![Screenshot7](./screenshots/7.png)

<br>
<br>

## Exploitation

Now that I had reached this point and had a better understanding of how our vending machine works, it was time to think about how to get the item __C8__ aka our flag.

We can only charge our user with a value of 1$ and we need 13.37$.

At first I thought maybe __Type Juggling__ somewhere or tried to top up the credit of user B from user A.

Tried to change the coupon code somehow and searched for another coupon code somewhere.....nothing.

I was in the Rabbit Hole again for a short time and then it occurred to me.

I'm currently reading a Bug & Bounty book that recently talked about.....Maybe it's a __RACE CONDITION!!!__.

Since race conditions are a popular target for everything to do with payment transactions, voting and auth bypasses, this would fit our challenge here.

A race condition occurs when multiple code sequences simultaneously access and manipulate shared resources without being synchronized.

As a result, the execution order may be unpredictable, leading to unexpected behavior.

This can be caused by sending requests in quick succession at the same time.

Now, how do we do this?

I first tried it with a simple curl-command which should recharge the voucher credit in a bash-loop.

However, this did not work: => `{"message": "This coupon is already redeemed!"}`.

The Burp Intruder didn't help me either, but I found an interesting Burp Extension which helped me: [Turbo Intruder](https://portswigger.net/bappstore/9abaa233088242e8be252cd4ff534988).

So I installed the extension in the __BApp Store__ under the __Extensions__ tab.

![Screenshot8](./screenshots/8.png)

It took me a while to understand how the extension works.

I sent a curl request to load a coupon and passed it through Burp using the `--proxy` flag.

In Burp I then sent the request to the __Turbo Intruder__.

![Screenshot9](./screenshots/9.png)

Well, the turbo intruder was something new for me and works with custom python scripts.

I did some research and found this helpful bug & bounty [Report](https://hackerone.com/reports/759247) for a race condition also found and used the python script for the intruder and customized it for my webchallenge.

I started the __Turbo Intruder__ and lo and behold!

![Screenshot10](./screenshots/10.png)

Apparently the account was topped up several times in a row....now I was excited!

For comparison, the usual message followed: `This coupon is already redeemed!`

![Screenshot11](./screenshots/11.png)

Anyway...now I bought a cheap product to check my balance and saw in the response that the account really has a higher balance.

![Screenshot12](./screenshots/12.png)

Hoooray, now I can afford the flag!

__Command:__

```
curl -v -X POST http://94.237.53.58:50782/api/purchase -H "Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyXzljNTk5Y2UyOWEiLCJpYXQiOjE3MDkyNjk3MjR9.9Dfcq35Sq2Gkp6R1gH2DE7Dzl9gJk13PYS1IeekE1dQ" -H "Content-Type: application/json" --data
```

__Flag:__

![Screenshot13](./screenshots/13.png)

<br>
<br>

## Conclusion

A really very cool web challenge as I would like to say again afterwards and for the future I would like to leave a few links and external resources here that helped me and maybe will help again in the future when I have to deal with it again.

<br>

__Used Turbo Script:__

[Used Turbo Script](./turbo_script.txt)

__HackerOne Report & Payload__

https://hackerone.com/reports/759247

__Burp Race Condition Labs:__

https://portswigger.net/web-security/race-conditions

__Hacktricks Race Condition:__

https://book.hacktricks.xyz/pentesting-web/race-condition
