---
layout: ../../layouts/BlogLayout.astro

title: That time when OOP and Microservices had a baby
---

### A tale about wasted labor, compute, and other horrors associated with this endevor

Many moons ago, I used to work at a company, the TLDR for the purposes of this article is that we were offering an API that you could use to interface with a number of other service providers. Think ordering plane / train tickets with many providers, or shipping packages.

At the beginning, the project was a quite large Django monolith (Python). However, problems began to arise. One to many times someone pushed some faulty code crashing everything, one to many times the upstream API returned some total garbage that we were not expecting (like a successfull response with a status 500) and causing everything to crash again -_-.

This is a huuge problem, management said. It must be adressed immediatelly! Hence a group of managers convened a meeting,
they thought, and talked for many hours, many days even and one day someone said the magic word, the solution, the holy grail.

## MICROSERVICES !!!!
Y'all heard about Netflix ? (I heard The Primagen used to work there btw).
At Netflix they use microservices, lets try that. Everyone in the room nodded and agreed, impressed with the modern, and enterprise proposition. Time to stop being a startup after all.


(I wasn't a part of those meeting but it must have happened in that way)

Thus began a multi year project of extracting each implementation into its own stateless microservice. What could go wrong ?


### Beginning architecture

We had a abstract base class with that defined the main operations as abstract methods and some general shared helpers if necessary. Then each upstream api would get its own class inheriting from the base class and providing the specific implementations. Classic good ol' simple OOP, 1 layer of inheritance deep, nice and simple.


### How do we make it microservice though ?

Excellent question dear reader, Well first we need to write an API Contract, so everybody is on the same page. Perfect, lets create a new repository, call it cool-api-schema, design said schema, write down all openapi yaml files. Done.


Then well remember that base class ? It had some helpers, well we would still like to have access to that in the microservices. Not to worry, we will create a package, call it blueprint, and not only will it contain all the nice helpers, it will also be the literal blueprint for the app. Define endpoints, types, core validations etc.

Perfect, progress is happening blazingly fast. Lets implement a first test service. Since the blueprint package provides all the necessary setup, the only part to do was migrating the code into the new repo, clean it up a bit, and deploy.

After connecting the main backend to use the newly created microservice everything worked, everyone got a pat on the back, and now we just have to do the grunt work of migrating everything nice and tidy into its own repository, and connect it one by one.


## The hidden costs

Well ...................... No

You see the amount of these microservices in the end was 100+. So every time these pesky sales / product folks would ask for a feature well, probably we need to send some more data first. Thats easy we update the schema and the blueprint model definitions, update the dependecies, and implement the features. That would be easy if not for the fact that that there were 100+ of these, so for a single engineer to update blueprint versions for every package would mean.

1. clone repo (if not cloned)
2. cd into it
3. pull
4. update the dependency
5. update the lock file
6. adress whatever has changed (model changes, utils, whatever)
7. create a pr
8. merge
9. and then repeat 100+ times !!!!!!!!!

Add to that testing requirements (because ofc during the migration process nobody cared about writing good tests, and at the time people doing the migrations were mostly early mid devs if not juniors), so what ended up happening is that completing dependency updates and non feature blueprint updates became a QUATERLY GOAL ALMOST EVERY SINGLE FUCKING QUARTER. Hundredths of manhours wasted every single Q just to update dependencies alone.

If that wasn't bad enough there was a practically almost yearly tradition of k8s cluster migrations, adding all of that together a significant percentage of the teams time (at the end idk 20 or so people) was spent on fake work that didn't bring any tangible improvement to the product whatsoever.

## The infra cost

Although the human cost here is probably much higher infra is even more infuriating.
You see since becomming enterprise, each service had to adhear to deployment and k8s configuration guidlines.
For redundancy, there had to be least 2 pods, if we are doing scaling well we need an autobalancer in front, each sercvice type had to have its own DNS entry, and so on, and so on. All of that for a even the smallest traffic services that were getting 1 request an hour. ENTERPRISE.
