---
title: "Calculating Impermanent Loss on Curve’s StableSwap Pools"
date: 2022-09-22
authors: [ibrahim] 
editors: [giray, ugur]
draft: false
math: true
---

There are various published impermanent loss calculators, yet there is none for
Curve’s StableSwap pools. Because StableSwap pools work with a different
methodology and are based on a different formula, traditional impermanent loss
calculators do not work on Curve’s StableSwap pools. As Dives Research, we have
developed an open-source app to calculate the impermanent loss on StableSwap
pools. We are not affiliated with Curve Finance in any way and developed the app
voluntarily. This article explains how we calculate the impermanent loss on
StableSwap pools. 

# 1. What is Impermanent Loss?

Impermanent loss is one of the risks of being a liquidity provider in certain
DeFi protocols. It is the difference between the value the tokens would have if you provided liquidity and the value the tokens would have if you held them without providing liquidity. 

When you provide liquidity to a pool, you allow people to trade between the assets you have provided. When the value of an asset increases, people will use your liquidity to buy that asset. Therefore, the amount of the asset that gained value will decrease, while the amount of the asset that lost value will increase. This means the combined value of the assets would be higher if they were not in a liquidity pool. Therefore whenever a price change occurs, there will be an impermanent loss.


# 2. How to Calculate It?

Let us also explain it with an example. Suppose that you have two tokens, A and
B. Right now, A’s value is 1 USD, and B’s is 2 USD. You have 50 A and 25 B. You
provide liquidity to the A-B pool on Uniswap.

When the value of A increases to 2 USD, people will use your liquidity to buy
more A. Therefore, you will have less A. We can calculate the amount of A you will have as follows:

> Let $a$ and $b$ denote the amount of A and B in the pool.  
> $a b = k$ where $k$ is a constant.  
> Initially, $a = 50$, and $b = 25$. Therefore $k = 1250$.  
> When the price of A is equal to 2, the pool will balance at $a = b$ because A and B have the same price now.  
> Therefore $a = b = \sqrt{1250} \approx 35.35$  

This means that after the price change, we will have approximately 35 A and 35 B. Multiplying the tokens with the price of 2 USD, we will have
approximately 141 USD  worth of assets. If we held the tokens separately, we
would have 50 A and 25 B, making 150 USD worth of assets. The difference, 9 USD, is the impermanent loss. 

# 3. What About Curve’s StableSwap Pools?

As we have seen, to calculate the impermanent loss, we need to calculate two things:

> (1) The value of tokens if they were not deposited into the pool  
> (2) The value of tokens if they were deposited into the pool

Calculating (1) is simple. Calculating (2) is not so much. 

In a regular Uniswap liquidity pool, the relation between the prices and the amounts of tokens in the pool is simple. Token A’s price in terms of Token B equals the amount of Token B divided by the amount of Token A. For instance, if there are 10k A and 20k B tokens, A’s price in terms of B is 2. So you need to pay 2 B to get 1 A. 

Curve pools do not work the same way. The relation between the prices and the token amounts is more complex. For instance, currently (30 August 2022), the amounts of tokens in Curve’s stETH - ETH pool is as follows:

ETH: 218,369.41 (27.25%)
stETH: 583,116.99 (72.75%)

But the exchange rate of ETH/stETH is 1.02783752.

So we need to find a way to derive the ratio of token amounts from the ratio of prices. How can we do that?

# 4. Understanding Curve’s StableSwap Invariant

We need to find the relation between the amounts of tokens and the prices of tokens. To do that, we have to look at how Curve’s StableSwap calculates prices. Curve uses the following formula to calculate the exchange rate of the tokens in a StableSwap liquidity pool:

$$A*n^n \sum_{i}x_i + D = A * D *n^n + \frac{D^{(n+1)}}{n^n\prod_i{x_i}}$$

This is the most general form of the formula. We will examine only the pools
with two tokens in them. $n$ is the number of tokens in the pool. So we set $n =
2$. $x_i$  is the amount of the $i$-th token. Since we are dealing with the
pools that have two tokens, we have $x_1$  and $x_2$. So, the formula for a
two-token StableSwap pool looks like this:

$$4A(x_1+x_2) + D = 4AD + \frac{D^3}{4x_1x_2} \text{ (Eqn. 1) } $$ 

$D$ and $A$ are constants. $D$ is similar to the constant $k$ in Uniswap pools. It is defined as the total amount of tokens in the pool when the pool is balanced. Unless there is a deposit to or a withdrawal from the pool, $D$ does not change, just like $k$ in Uniswap. 

The important thing is that the ratio of $x_1$ and $x_2$ in a pool does not
depend on $D$. We can see this by creating a graph of this simplified equation.
In this graph, changing $D$ moves the graph on the $x = y$ line, but it does not
change the shape of the curve.

![](https://media.giphy.com/media/Osi9vlm6C9nPD6zzbm/giphy.gif#center)


For a more rigorous demonstration, suppose we multiply $D$ with a constant $v$.

$$ 4A(x_1+x_2) + Dv = 4A(Dv) + \frac{(Dv)^3}{4x_1x_2} \text{ (Eqn. 2) }$$

Take any $x_1$, $x_2$ that satisfies **(Eqn 1)**. We can show that $vx_1$,
$vx_2$ satisfy **(Eqn 2)**:

$$4A(vx_1+vx_2) + Dv=4A(Dv) + \frac{(Dv)^{3}}{4vx_1vx_2}$$

Equivalently,

$$v(4A(x_1+x_2) + D) = v(4AD + \frac{D^{3}}{4x_1x_2})$$

When we simplify by dividing both sides by $v$, we get **(Eqn 1)**. We know
$x_1$, $x_2$ satisfy **(Eqn 1)**, therefore $vx_1$, $vx_2$ satisfy **(Eqn 2**).

We have seen that when we multiply $D$ with a constant, $x_1$ and $x_2$ values multiplied by the same constant satisfy the equation. Therefore the whole curve is multiplied by the same constant. So, $D$ does not define the distribution of the pool. This means when we want to calculate the ratio of $x_1$ and $x_2$, the value of $D$ is not important. Therefore, we can set $D = 1$ to make the calculations easier. So, we have the following formula:

$$4A(x_1+x_2) + 1 = 4A +\frac{1}{4x_1x_2})$$

Lastly, $A$ is the defining constant of the pool. It determines the shape of the
curve. When $A = 0$, the StableSwap pool works exactly like a Uniswap pool. When
we make $A$ greater, the middle of the curve gets straightened. We can see this
in the following graph:

![](https://media.giphy.com/media/LVYuej8rk9aHbPlMZf/giphy.gif#center)

For a more detailed understanding of how the StableSwap formula works, refer to [Curve’s StableSwap Paper](https://curve.fi/files/stableswap-paper.pdf).

# 5. How Price is Calculated on StableSwap

For now, what we need to understand is how to calculate the price using this
formula. As you may realize, once we set $A$, $D$, and $n$, we are left with a
formula with two unknowns: $x_1$  and $x_2$ . This means we can treat $x_2$ as a
function of $x_1$. Let $y(x_1) = x_2$, and let $x = x_1$. We want to calculate
the ratio of the change in $y$ to change in $x$, which will give us the relative
price of the tokens in the pool.

Suppose that $x = a$, meaning that the amount of the first token in the pool is
$a$. Therefore, the amount of the second token in the pool is $y(a)$. How can we
calculate the exchange rate between these two tokens? Suppose we have $h$ amount
of the first token, and we want to exchange it for the second token. We can add
$h$ to $a$, so $x = a + h$, calculate $y(a + h)$, and get the difference between
$y(a + h)$ and $y(a)$. This gives the change in the amount of the second token
in the pool, which is the amount that we receive in exchange. So we can
calculate $$\frac{y(a + h) - y(a)}{h} \text{,}$$ which gives the exchange rate for $h$
amount of the first token. If we want to calculate the exact price at $x = a$,
we should take the limit of this ratio as $h \rightarrow 0$.
$$\lim_{h \to 0} \frac{y(a + h) - y(a)}{h}$$
This limit is the definition of the derivative! This means the price at an exact point in the curve is the absolute value of the curve’s derivative at that point. We need the absolute value because this derivative is always negative.

![](https://i.ibb.co/sy0rnt7/final1.png#center#resize-half)

# 6. Calculating Impermanent Loss on StableSwap Pools

Now we know that the price at a point in the curve is the absolute value of the derivative at that point. As a reminder, our simplified StableSwap invariant looks like this: 

$$4A(x+y) + 1 = 4A + \frac{1}{4xy}$$

So, if we take the derivative of the simplified StableSwap invariant, we get:

$$4A(y' + 1) + \frac{1}{4}(\frac{1}{x^{2}y} + \frac{y'}{y^{2}x})= 0$$
$y'$ is the price of $x$ in terms of $y$, with a negative sign. 

As discussed in **Section 3**, we need to find a way to derive the ratio of the tokens in the pool from the price of the tokens. We now have the derivative and simplified StableSwap formulas, so we have two equations with the unknowns $x$ and $y$. This means when we set $y'$, we can calculate $x$ and $y$ values that satisfy both equations. Therefore we can calculate the ratio of the tokens in the pool.

Instead of solving the equations, we can calculate the intersection of the two equations with whatever precision we want. Finding this intersection is what our calculator does. After finding the values $x$ and $y$, we can easily calculate the impermanent loss.

Suppose we want to calculate the impermanent loss in a StableSwap pool. We have
two relative prices: $p_i$ (initial price) and $p_f$ ([final](final) price) If we deposit
our tokens when the price is $p_i$, and the price changes to $p_f$, how much is the impermanent loss?

We can calculate the values of tokens if they are not deposited to the pool by
calculating: 

> $x_{p_i}$ (the amount of the first token when the price is $p_i$)  
> $y_{p_i}$ (the amount of the second token when the price is $p_i$)

Then we can calculate the first value denoted by $V_1$ as follows:

$$V_1 = x_{p_i} * p_f + y_{p_i} $$

We can also calculate the values of tokens if they are deposited into the pool.
We first calculate 
>$x_{p_f}$ (the amount of the first token when the price is $p_f$)  
>$y_{p_f}$ (the amount of the second token when the price is $p_f$)

Then we calculate the last value denoted by $V_2$ as follows:

$$V_2=x_{p_f} * p_f + y_{p_f}$$

And the impermanent loss (IL) is 

$$IL = \frac{V_2 - V_1}{V_1}$$
