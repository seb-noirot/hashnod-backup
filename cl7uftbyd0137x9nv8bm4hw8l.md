# Javazone 2022

Finally back to [Javazone](https://2022.javazone.no/#/)!!! Biggest java event in Norway, 2 days (3 if you have workshop tickets), amazing speakers, crazy organisation, inspiring community!

I was there in 2019 for my first time and I was focusing on tech talks only. But this time, with current stressful context (COVID, war, Global warming, inflation), I also wanted to join some soft skill talks, to listen to people that share my interrogations, how they handle them, what are their conclusion on their action, etc.

## Soft skill talks
### [My career path](https://2022.javazone.no/#/program/ecada2e9-b0b1-466d-8f5d-5ceaf38e50d5)

During this talk, if I have to remember only one thing is that an engineer is not a junior manager!!!
Don't let them tell you that the only step after senior engineer is manager!
There are other way to stay in engineer world:

- staff engineer,
- principal engineer,
- distinguished engineer.

I will try to also keep in mind that a career is not a straight line and it is ok and normal to take turns, to come back, to jump over, to take a break, to focus on where we are.

### [My mental health](https://2022.javazone.no/#/program/2ddbb85c-f8d5-4287-b3d9-902c2d30c5fb)

This talk was mind blowing : I knew all, all was common sense, I apply nothing in my life!
We all see those position in LinkedIn:

> Search full stack senior engineer, DevOps, Java, Python, C#, React, Angular, Kubernetes, Docker, AWS, Azure, Google cloud, worked with high performance / distributed / critical systems, wrote 2 books, involved in community, amazing social skills, mentor, climb, sporty, speak 10 languages.

When this is the norm in our profession, one tends to overwork to learn all those things, do all those things, in hope to reach this Graal position!!! Until burnout. 

So now, I will try to apply few of his advices to release the pressure on my mind and enjoy who I am as an engineer, enjoy learning to grow as en engineer.

### Conclusion

Those 2 talks were the only one I followed. They either learned me new things, or make me realise I don't apply to myself what I would recommend other people to do. I think we are an industry focused on tech and evolutions and sometimes, to take a step back to focus on ourself is vital for our future.

There were more during the event and I will watch them also for sure when they are available on youtube.

## Pure tech talks

I chose tech talks mostly based on the stack I am working on:
- Java (Quarkus / Spring) in backend
- MySQL, Neo4J, Redis, S3 for the data
- React in frontend
- Event-driven (Kafka, RabbitMQ)
- Rest, gRPC
- TDD (JUnit 5), Acceptance tests (Cucumber), Performance (k6)
- Docker, Kubernetes
- Gitlab, Whitesource, Sonar
- AWS

The goals were:
- to (un)validate the way we use those techs
- to learn how to fully use those techs
- to learn about the future of those techs
- to learn about alternatives to those techs

### [ Event / Decision driven](https://2022.javazone.no/#/program/ec02e188-684b-4892-83b6-4014b4b02272)

During this talk, I learned that instead of event driven architecture, we should call it decision driven architecture. The reason behind that is that your process listen to event, but the decision you take based on the event is the logic of your process.

I also learned how to "recover" or "redo" a list of events. You can either replay all the events from a certain time but it can also be worth it to generate snapshots and fetch this snapshot and only replay the list of events from this snapshot.
In our stack, we are doing the snapshot manually while we could move this task to our Kafka for example.

To make it simple : our way of thinking is correct, our implementation could be improved.

### [K6 not only for REST](https://2022.javazone.no/#/program/3ac1dbc9-d768-4a90-87e3-7e74dab70f6a)

In our stack, we have different type of exchange between each elements:
- REST
- gRPC
- Kafka
- Database

At the moment, we only run automatic performance tests on our REST layer. With k6, we can also do performance test on other layer of our application like gRPC (not for the stream), Kafka, DB.
We could easily implement tests on those layer to validate:
- our configuration like topic replication, partitioning, ...
- our configuration in terms of setup : CPU, memory, network
- calculate our burst limit for each component of our stack

One final step would be to have those results pushed to Grafana, and of course, there is a plugin for that

### [gRPC vs REST](https://2022.javazone.no/#/program/974685d0-fd4a-4466-8d09-7d0aea9820ad)

This talk is great for someone doing REST api, and wondering what is gRPC about. I like the approach that gRPC is not there to replace REST everywhere but can solve performance issue in term of (de)serialisation created by json format, is recommended for backend to backend communication (who need a REST with JSON in backend to backend... ok I have a lot of those...)

So something we have to change in our stack, or start to apply in our future development. 

### [Release note / Changelog](https://2022.javazone.no/#/program/94a7f915-16cb-444d-8d37-968ea2f4c0d7)

During this presentation, we have seen the whole process between :

- I have a new version
- Consumers use this new version

This process will contain several steps like creating the release version, make it available, create a release note / changelog, package the new version (homebrew, sdkman,...), notify users (slack, emails, ...) and all that with only a configuration file.

At my level, the release note / changelog is something that we have to implement for several reason:
- in an 2 week sprint tempo, with a microservices stack, it is difficult to know what is deployed.
- in case of a rollback on a service is needed, we know to which state we go back and what are the feature not deployed (an thus the impact of the rollback on the rest of the stack)
- easier communication with the rest of the team / consumer of the services.

### [Apocalypse Now](https://2022.javazone.no/#/program/ddc4c425-9a4a-44d9-9d88-59a6b29bf87f)

This presentation rings a bell. Recently we had some strange behaviour on our system, and tech support didn't managed to solve it by themself:

- lack of documentation
- lack of training

Our documentation is written by us, developers, people who know about our system. It obviously contains shortcuts in the explanations, misses several why / who / how / when / what details, is tested by us only.

Where does it lead us to ? A disaster!!!! (or a Sunday morning call from tech support). A shame as we have the knowledge but failed to share it. The recommendation to prevent this is to do regular training by putting people in front line in a fire situation and see how well they perform with all the information / documentation they have or can find and take action from there!

We are a software company, with a passion for games, board games, video games. I would like to introduce to test our processes in a "fun and interesting" way : a Dungeon & Dragons about our apocalypse!!!

### Conclusion

I went to other talks that did not trigger any alarm for me but the list above are talks that even a tiny part of them made me realise that I am missing something, that I could do something in a better way, that I could use some tools their limits. 



## Conclusion on Javazone

I learned a lot in both tech and soft skill and now I have the responsibility to share them with my team and see what we can take, what we can plan, and see if we can grow from it until next Javazone!!!!






