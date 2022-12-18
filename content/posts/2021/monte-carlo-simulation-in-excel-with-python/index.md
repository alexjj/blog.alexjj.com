---
title: "Monte Carlo Simulation in Excel with Python"
date: 2021-11-25
summary: "Simulating the possible with python and excel"
tags: ["Python", "Excel"]
---

There are quite a few blogs that show you how to do Monte Carlo simulations using python. A very accessible one is from [Practical Business Python](https://pbpython.com/monte-carlo.html). Always a great site for python in the workplace for non-devs. However, I couldn't find one that did just what I wanted, namely, take a model I'd already built in Excel and then apply some randomness and plot the S-curve (cumulative distribution function).

Maybe if I'd searched more I'd find one but instead I just got on with it. I also used to have some software at work that did this for me, it was an add-on to Excel but work stopped paying for it and I found it hard to justify it (it's an Oracle product so no doubt pricey). Plus I always love an excuse to try and learn a bit more python.

## Background

If you don't know, Monte Carlo simulation is a way to capture uncertainity. If a friend is driving to your house, 50 miles away, and they travel at 50 mph it'll take then one hour right? Well, not always. Maybe they are keen on the gas and drive faster, or maybe there's an accident and they get stuck. You just never know. So you could use monte carlo to say that their average journey speed could be from 40-60 mph, then you randomly sample many many times from that range and work out the time it takes. Do it 10,000 times and you then have 10,000 results that you can draw estimates from, e.g. the average time. You can also plot the results and show the whole range.

## Tools

The main python tools to get this working is [numpy](https://numpy.org/) for the random sampling and [xlwings](https://www.xlwings.org/) for interacting with Excel. Like all python libraries, these tools do **way** more than just what I need. I also used pandas, seaborn and matplotlib. Probably unnecessary but they're quick to work with as I already know the gist of them.

Then of course there's Excel, which needs no introduction. I'd already built my deterministic model in Excel, where all my inputs were in one table that could be changed and then the outputs were in another to be read.

There's always going to be the choice of what do you do in python and what do you do in Excel. As I'd started in Excel I didn't want to waste time recreating stuff in python so all the calculations are in Excel. This is slow (but still fast enough for my use case). Depending on how advanced or complex your model becomes it may be better to move more of it to Excel. The trade off with this is now only you can use it or understand it.

Before I forget, there's also VS Code. I use it at work as it's free, I can install it without Admin rights and IT department haven't questioned me on it (yet!), unlike PyCharm 🤔. I like to make use of the #%% cell divider in it. Partially as I'd been using this for a long time with Spyder (before VS Code existed) and also as it seems like a better way to work vs. notebooks. At least for me.

As always the documentation of the libraries is a great place to start - often the best place before you jump to tutorials.

## The Randomness

Your uncertainity takes shape in many different ways. Sometimes is a continuous range, like our speed example before, or it's discrete, like picking your starter Pokémon. In fact whatever the uncertainity is, numpy has a [method](https://numpy.org/doc/stable/reference/random/index.html) to help you represent it. I actually think working out what your uncertainity looks like is the hardest part of all of it. Perhaps you could even have uncertainity on the uncertainity...Generally you want to ask an expert, maybe yourself, maybe someone else in your company, or whatever you can find online that seems reasonable. You can, of course, just try different ones in your model and see if it actually makes a difference to the decision you're trying to make.

I ened up using `choice` and `uniform` distributions. Choice being my starter Pokémon, and uniform distribution being a range of numbers but they're all equally likely to be picked.

My choice was actually choosing a range of numbers in Excel which would all change depending on what choice was picked.

## The Code

So in all its glory:

```python
import pandas as pd
import numpy as np
import seaborn as sns
import xlwings as xw
import matplotlib.pyplot as plt

sns.set_style('whitegrid')

# Setup
excel = r"myexcel.xlsx"

wb = xw.Book(excel)
sheet = wb.sheets['Control Sheet']

rng = np.random.default_rng()

iterations = 1000
POWER_LOWER = 100
POWER_UPPER = 120
CAPITAL_LOWER = 10000
CAPITAL_HIGHER = 30000
CO2_OPTIONS = [1, 2, 3]


npv = []

for i in range(iterations):
    power_price = rng.uniform(low=POWER_LOWER, high=POWER_UPPER)
    capital_cost = rng.uniform(low=CAPITAL_LOWER, high=CAPITAL_HIGHER)
    co2_price = np.random.choice(CO2_OPTIONS)

    sheet.range('B3').value = power_price / 80
    sheet.range('B4').value = capital_cost
    sheet.range('B9').value = co2_price
    npv_new = sheet.range('B7').value

    npv.append(npv_new)

results = pd.DataFrame(data={'NPV10 £MM': npv})
P90 = results.quantile(0.9)[0]
P50 = results.quantile(0.5)[0]
P10 = results.quantile(0.1)[0]
EV = results.mean()[0]

sns.displot(results, kind="ecdf")

print(f"EV: £{EV:.1f}MM, P10: £{P10:.1f}MM, P50: £{P50:.1f}MM, P90: £{P90:.1f}MM")
```


I made a few controls at the top so I could adjust the ranges of things. Fairly basic stuff otherwise, which I'm sure can be done better but was good enough for me. Being so simple makes it pretty easy to understand, even if you've never used these libraries before.

The whole thing is a loop that does the random sampling, insert those values into excel, excel then recalculates, then it takes the answer out and appends it to a list. Repeat that `iterations` times and there's my data.

I create a pandas dataframe (probably lazy and it could be done in numpy but as I said, lazy.) from the list and then calculate some metrics from it. The P values are commonly used at work to express the range. Where P90 meaning 90% probability that the value is ≤ X. Or often viewed as most optimistic result, with P10 therefore being the most pessimistic result. Actually, I've seen companies define these in reverse so check before you confuse people.

Seaborn has a handy empirical cumulative distribution function (ecdf), that saves me thinking anything whatsoever (that's the whole objective whilst at work right?). So I make one of those for the PowerPoint, and print out the final numbers in a nice line.


The other nice thing with VS Code is the data viewer. I can save the plot to a file and copy and paste the table of results straight from it without having to write it to a file first. Tiny things like that make life so much easier.


## Conclusion

So pretty straight forward to make your deterministic Excel model into a fully probabilistic model capturing all the uncertainity and demonstrating all the potential outcomes that can happen. For free, in about 30 minutes of work.

I think probabilistic models should be used more often. Very rarely is it that everything is always known exactly and whilst your calculation isn't wrong, it's only one possible outcome. Better to give your decision maker (or yourself) a range of outcomes than just one. Tell them "they're buying the curve". Then hope the final result actually lands on the curve 😅
