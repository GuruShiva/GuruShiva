---
layout: post
title: Bobby Drop That GraphQL Schema
image: /img/graphql/graphql.png
---

Being part of an active infosec community is one of the best things where you get to learn some new technologies, tricks, and also pass on the knowledge to eager beginners or even the seasoned pros. 

Whenever I have someone come up to me asking "How do I get into the field of Information Security?". One of my top 3 suggestion is to join a  infosec community whether it is an online one such as the popular [NetSec Focus](https://www.netsecfocus.com), a local infosec meetup or even your local [Bsides Event](http://www.securitybsides.com/w/page/12194156/FrontPage#PastPresentandFutureBSidesEvents).


It was during one of these infosec community meetups that a bunch of us got a chance to test an upcoming innovative platform for security misconfigurations, vulnerabilties etc. Now before I proceed, please read the below mandatory disclaimer

![DISCLAIMER](/img/hush-read-the-disclaimer.jpg){:class="img-responsive"}

**DISCLAIMER:** *ALL THE TESTS DEMONSTRATED BELOW WERE DONE WITH THE EXPLICIT PERMISSION FROM THE OWNERS OF THE APPLICATION. USE THIS ARTICLE FOR EDUCATION/INFORMATIONAL PURPOSES ONLY. THE AUTHOR AND OTHER PEOPLE MENTIONED IN THIS BLOG DO NOT CONDONE ANY ILLEGAL HACKING ACITVITIES. ALWAYS PERFORM SECURIY TESTS ONLY ON APPLICATIONS,NETWORKS OR ANY EQUIPMENT THAT YOU OWN OR HAVE RECIEVED EXPLICIT PERMISSIONS FROM THE OWNER*.

Ok, now for a little tale on how [Bobby](https://xkcd.com/327/) or in this case Hummus(me) dropped a GraphQL schema [skip to GraphQL part](#graphql) all with the help of some great infosec folks that helped me grow in this field.

It was a friday evening after work when a group of us joined for some pizza, burbon and some good stuff of vulnerability testing. The application we were to test is the up-to-date version which will be pushed to production soon which makes it even more exicting if we found any vulnerabilties. 

After the introduction of what the software plaform does, a high-level point-of-view of its technical architecture and who the market is (which I cannot go into any details), we were told even though the client-application talks to various API endpoints, the current version is only available for iOS while development is being done for other platforms. Ok, thats a bummer. Because none of us were prepared for testing an iOS app directly. However, that did not stop us from coming up with plans to test the application through the iOS platform. The creative juices started to flow.

We were all coming up with ways on how to test and find out how this iOS app communicates with the API endpoints. I was struggling with how to get the iOS app on an iphone to proxy the traffic through Burp Suite so I can inspect the traffic. However, iOS being one of those technologies decided to give me a hard time mostly because I was not prepared to test an iOS app that evening. I have almost decided to call it an evening when Don Burks (one of the amazing software developers, devops, security guy) whom I have had the honour and pleasure to meet through the same info community and one his colleague (also amazing), both who played major roles in the application development decided to give us some of the Internet accessible API endpoints. The hunt begins again. 

One of the API endpoints happens to be a GraphQL endpoint which the iOS app communicates with to retrieve/send database queries etc. As fate would have it, I decided to test the GraphQL endpoint using POSTMAN application to perform GET and POST requests and see what happens. At this point, I am just trying mostly my luck and see if I hit anything. Some of my more experience peers were also fuzzing around with the GraphQL endpoint along with trying to figure the whole iOS app as well so chances are one of us might actually find something (hopefully?).

This also happens to be my first time testing out GraphQL and its magical capabilties so I decided to spend a chunk of time using Google and hitting up blogs etc to understand what [GraphQL](https://medium.freecodecamp.org/a-beginners-guide-to-graphql-60e43b0a41f5) is (which I am not going to talk about here) and how other security experts have tested GraphQL implementation for misconfiguration or vulnerabilties like NoSQL Injections. I found this short amazing blog -> [graphql-security](https://blog.doyensec.com/2018/05/17/graphql-security-overview.html). Skimming through it I now have enough information to fuzz this GraphQL endpoint

## The Juicy GraphQL Part <a name="graphql"></a>

I launch POSTMAN (an app that I now have more respect towards after that adventurous evening) and put in my first query 

![graphqlget1](/img/graphql/graphqlget1.1.png)

The Graphql endpoint returns this:

![graphqlgetresult1](/img/graphql/graphqlget1_result1.1.png)

oh look! what do we have here. In the day and age of smart recommending engines, GraphQL is also being helpful. It tells me that the `Email` query does not exist but suggests that I might be looking for `emailExist` query. Am I looking for the `emailExist` query? why yes GraphQL, thats exactly what I meant. At this point, I am getting even more excited. What's more, I didnt even have to use authorization tokens to query this. 

![excite1](/img/excite1.gif)

My new query to the GraphQL endpoint was

![graphqlget2](/img/graphql/graphqlget2.1.png)

and then I get this from the endpoint

![graphqlgetresult2](/img/graphql/graphqlget2_result1.1.png)

![excite2](/img/excite2.gif)

Ah. The query `emailExist` query requires a string. Well of course. I am querying whether an email address exist or not. There was a test user with a test email address which was mentioned during the initial briefing of the application, so I decided to check for that email address. At this point, I have already mentioned the finding to Don who went something along the lines of "Oh interesting, that was not supposed to happen." He then goes off on a hunt of his own as he knows more about GraphQL than I do. My experience with GraphQL started only 20 minutes ago. 

I return to my POSTMAN command center (yes that is a temporary made up title) and run my next query

![graphqlget3](/img/graphql/graphqlget3.1.png)

which returns the new result

![graphqlgetresult3](/img/graphql/graphqlget3_result1.1.png)

Ding! Ding! Ding!, we got a winner!

![excite3](/img/excite3.gif)

One can imagine the bruteforcing potential with this. Now, the important thing to remember is that this is not a vulnerability or a bug within GraphQL. It is an actual feature (I smirked writing this part). The feature is called 'Introspection' which you can read more about [here](https://graphql.org/learn/introspection/). It helps the developers during the software development phase to test out their API's data queries etc. The even more important thing to remember is to turn off such features before pushing your application to production. 

As I was finalizing my findings with my other peers, Don goes full commando on the GraphQL query to test how worse can this really get

![graphqlget4](/img/graphql/graphqlget4.1.png)

which basically drops most of the data schema of the application if not all

![graphqlgetresult4](/img/graphql/graphqlget4_result1.1.png)

Well then, like they say, the rest is history. Don takes a note to turn off 'Introspection'. I was pleased with myself and no longer disappointed not being able to test through the iOS app.

I hope you guys enjoyed reading this as much as I enjoyed testing the GraphQL endpoint. Until next time, stay safe and have fun.






 