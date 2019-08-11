---
title: The Phone Screen Works Both Ways
excerpt: A phone screen is typically used to screen applicants for a job, but it gave me enough information to not want to work for the company.
tags: business work
categories: Business
classes: wide
---

I recently applied for a position through a recruiting service. I jumped through the requisite hoops and was told that the prospective employer was very excited to talk to me.

The standard procedure for this employer is to give candidates a simple phone screen before arranging an interview. Personally, I feel that having one person make a decision about whether or not I’ll “fit well with the team” after 10 minutes on the phone is ludicrous, but that’s the game I had to play.

During the phone call, I was asked several generic questions about Java. I gave what I felt was correct answers to the questions, but the interviewer wasn’t happy with my results. Every time I answered a question, I heard “Well, I was looking for…”, with the expected answer simply being a different wording of what I had said.

I chalked this up to the interviewer simply being used to different terminology than I for the same concepts, so I was a bit surprised when I wasn’t called in for an in-person interview. It wasn’t until a few days later that I realized that the answers I was expected to give told me quite a bit about the company, and that I wouldn’t have been happy working there had I been offered the job.

The particular question that stuck in my mind was a simple one. “What are some of the benefits of the introduction of generics in Java, especially in collections?”

I gave the answer that pretty much anybody familiar with Java generics would give: compile-time type checking, no need for casting objects, etc. What I got from the interviewer was “Well, I was looking for the fact that you don’t have to do `instanceof` checks all over any more.”

On the surface, this may seem like another way of saying what I said, but it’s actually quite different.

Before generics, you would need to cast objects to the proper class when retrieving them from a collection. This does not mean that you would be using `instanceof` to do this. The only situation I can think of that requires using `instanceof` with a collection is if there are a lot of heterogenous objects in the collection.

There are limited situations where storing different object types in the same collection makes sense. Most of the time, however, this is a sign of either lazy development or not understanding the collection mechanism very well.

It’s generally a good idea to only put one type of object into a collection. This makes it much easier to work with and avoids any need for `instanceof` checks. By “one type of object”, I don’t mean that all of the objects need to be the exact same class. Chances are good that all of the objects being placed into a collection have some relationship; perhaps they will all implement the same interface.

I may have completely misinterpreted the interviewer, but I have a very strong impression that he and perhaps others on his team are used to using collections as grab bags of widely different things. Personally, I wouldn’t want to maintain that code. I can hope that the addition of generics to Java has forced the interviewer to change at least one practice for the better. I’m glad I wasn’t called in for the job — who knows how many more “well, at least it works” practices are being followed? I’ve got enough of those at my current job; I don’t need to learn a new set.
