---
title: "Travel by Bicycle on Public Transport"
author: "Cristian Bourceanu"
date: 2025-05-10T20:16:00Z
categories: [bike, train, public-transit, app]
---

# Travel by Bycicle on public transport

Welcome fellow cyclist or soon to be! To cut it short I am pouring here my ideas
about improving the way we can plan travels when it comes to minimising our
carbon footprint and enjoy maximising our time for what matters, rather than
endless hours spent in airports and driving a car, which let us be honest, it is
lost time that you will never get back.

## Short story about me

::: {.callout-tip}
If not interested, please jump to the technical part in
@sec-application-requirements.
:::

Even though I didn't have a clear idea about the impact climate change had when
I used to be in high school, I decided that I don't need a driving licence to
contribute to green house gas emissions, while all my other classmates got one.

Then later in university, cycling became my main way of commuting while I was
living in Oxford, which is a very welcome city to start cycling since the centre
is restricted to bicycles, buses and taxis, but few cities in UK have this
benefit, unfortunately.
During my undergraduate there I lived in house with a Polish PhD student who was
cycling about 50 km every other day and pushed my to explore even more of the
surroundings, which by the way are very pretty.

After my graduation, I continued this style of life, while a component has been added to
the commute chain, travelling by train. I started working right away in a far
east town in London, Rainham , and initially I used to take the train and cycle
forth and back (60km total) for 2-3 days per week to get to my job. 
The GWR trains Oxford to London had bike space all the time available at
the time of day I was travelling, but once in a while it would happen that
, without a bike reservation, I had to implore the staff to let me on the
corridor, which I notice to be a hard rule not to be accepted any more.
I did this for 2 months, then I moved in Isle of Dogs and the commute shortened 
to 36 km total per day.

After one year in London, I moved to Cambridge for a new job and with this a
new lifestyle started. If you ever lived in this area you might know that the
area around Cambridge is one of the most boring in the UK. Because I really like
to explore the natural world, I started travelling, and hence is where my real
troubles with travelling on trains with my bike started.

### Where the fun begins

Normally the train Cambridge-London accepts bicycles, apart from the peak hours.
But if you want to travel longer distances, like going north to Peak District or
all the way to Scotland, there are a maximum of 4 bike spaces for 500-600
passengers train on LNER and Avanti, or none for Lumo. The train companies
don't have an incentive to improve the integration of cycling and travelling by
train, as for them there is no pecuniary benefit, but on the contrary, since the
bike spaces are free. Policymakers are probably better to discuss how
this problem could be resolved, but here I am going to focus on what is within
our power to make this activity easier.

Furthermore, not many coaches provide bike spaces either and for the one that
do, it varies if you need to book, or it is first come, first served.

### Moving away from airplanes. Is it possible?

Above, I have focused on the problem in UK as I live here, but the same stays
true for Europe. 

Last year, in the summer of 2024, because I decided to stop
taking any flights, I travelled from Cambridge to Romania on my modified 
single-speed Mango bike and back. However, due to time constraints, I had to cut
it short and take the train on a section. Well, it wasn't easy either to cross
to Europe on the Eurostar, or find a train that would accept bicycles on a good
chunk for my travel (about 600km).

::: {.callout-tip}
TODO: Add diagram of my trip, splitting public tranport and cycling paths such
that I can depict the problem better.
:::

This journey discouraged me if I can do this any further. If you travel
UK-Germany for example, that is not too bad, but if you go all the way to Romania, 
there are not many options, and the romanian trains are despicable.

### Added problem

Besides the admin hasle of trying to figure out what trains accept bicycles and
on what trains still have bike spaces available, there is also the
travelling salesman problem of what route you should take to get from point A to
B given a certain cycling speed. 

Now I live in Scotland and the problem amplifies even further since the
highlands is sparse populated, hence also the train stations and tracks are
fewer. So if you wanted to travel from point A to B, you have to check many
choices of how to combine cycling, with train, ferry or buses.

::: {.callout-tip}
TODO: Add example for one of my travels in Scotland.
:::


## The solution {#sec-application-requirements}

I spent to much time being upset or angry on the lack of options or the money
I've lost because I had to change tickets to allow me to travel this way. Hence,
the only way to move forward in doing what I like, to be free, to get to
any point in the world, while preserving the environment we have for generation
to have, is to try to ease the way such multiple green choice transportations
can be combined and to make sure the travel is possible that way.

Solutions to this problem vary depending what your focus is, as a journalist you
might write many articles to bring awareness, as a political activist you would
try to push policymakers to move the game in that direction, if you are involved
in the government or have an official political position, you could change the
game from within. I am none of these, but what I am good at is being an
engineer and as an engineer I would like to have a tool that can ease the way I
can live given the rules of the game I am playing.

Hence, the problem I described above is two-fold:

- Travelling salesman using multiple choice public transport and freestyle travel (cycle or hiking)
- Check the availability of bike spaces on the public transport choices

### Breaking down the problems

#### Travelling salesman

While the first problem might be achievable for only public transport as the
nodes need to be connected, if you cycle the amount of choices for each station
increases very fast with the radius you are willing to travel. 

But let us assume first that we only have public transport to care about. For
the national rail, there is a public API you can use to query the routes, but
for buses and ferries, you will need to check with each company individually.

Ideally, you would have an API for each one of these that can give us a good
confidence about the results, but what we are left with is building a
heterogeneous querying or scrapping the web for information, which can be very
expensive, hence some memoization is required.

The long term solution could be to commercialise such a product and pay
brokers that can give you reliable information about the routes.

Moving back to adding the cycling or hiking in the forefront, services as Google
Maps are already hard to achieve to integrate public transport with short walks,
extending the range of reaching the public transport station, makes it much more
computationally complex as shown in @fig-exponential-travelling-salesman.

::: {.callout-warning}
Increased difficulty to find best A->B route for this problem needs to be shown
in a visual example.
:::

#### Bike on public transport

As we discussed above, the transparency of these transportations method is an
issue, and becomes quite problematic with bike spaces, as there is no
information on the available API in the national rail about how many bike spaces
are available on the specified route.

The way I had to do it so far was to trust Trainline new added feature about
bikes, which turned out not to be very reliable as reason for why I am writing
this blogpost. 

It seems [PlanarNetwork](https://github.com/planarnetwork) distributed transit
API might have some fields describing bike allowance, but this needs to be
investigated and potentially talked with the developers if we can move this
forward.

Alternatively, it would to have a bot developed that tries to book a train
ticket for a route on the official train operator website and check if a bike
reservation can be made. This is probably quite tricky to make, but only
certain solution that I see moving forward for now.


### Discussion

There are many requirements and potential solutions that need to be discussed,
above I only wrote the very basic high-level description of the problem that I
want to solve.
If you are interested to discuss more about this and have ideas to contribute
please contact me.


## Implementation plan

My focus to start with will be trying to sort the travellings-salesman of bike +
train space and see if I can ease up how I can explore Scotland using this
lifestyle.

Once this is completed, it will give me more certainty that I can look into bike
reservations on the train.

Finally, adding more options for public transit should be added easily, given
that I leave enough flexibility in how the routes are structured and described
for the trains.
