---
layout: post
title: The Art of Workflow
---

Software development is a team sport. Writing an entire code base as one person is surely convenient at times, but it is not realistic to be a lone developer writing the entire code base at a company.

At the beginning of October, the instruction team at Nashville Software School divided my cohort into groups. We were assigned group projects for the remainder of our time at the bootcamp. My team put our heads together and tried to figure out the best way to manage a long-standing team project. We spent two days of classtime planning how we would manage each project. We brainstormed our process from start to finish.

Newer developers who have not worked in a team environment before might wonder why a process matters on a team. A developed workflow allows your team members to work independently on separate issues and also come together to overcome problems that one can't solve alone.

> Your workflow is the standardized way to effect meaningful contributions to your team's goals.

One team's process for getting work done might look different from another team's system. Our team found what worked best for us. There is no right or wrong way to managing your workflow; however, I think that all effective workflows share the following principles:

1. Team members must agree on the project requirements.

Every team member should have an understanding of what the project entails. It's ok if a requirement is ambiguous initially. It is the responsibility of the team to ask the product owner or manager to clarify what he/she expects for a certain requirement. The team should bring specific questions to the product owner to make the requirement as clear as possible. User stories should be defined and unambiguous as well. Without a clear understanding of the requirements, a team will not have a clear idea of what the finished product is.

2. Work should be divided into small chunks so that each chunk can be completed independently of other chunks when possible.

When beginning to work on a project, team members should be able to assign tickets to themselves without having to wait on work to be completed by another team member. There are exceptions to this, however. Obviously, if a team member wants to implement a "User Profile" page, a feature should already be implemented that allows users to log in. If too many chunks of work are reliant upon other chunks to be completed, a bottleneck effect occurs in the team. This creates an environment where only one person is only working at a time, since each feature is dependent upon another and can't be implemented until that developer has finished his/her task. Smaller pieces of work allow for your team to quickly contribute working pieces of code.

3. Work should be divided into tickets, which act as "to dos" for the team. Each ticket should be a detailed note of the exact work that needs to be done to implement a feature.

Amiguous project tickets are the quickest way to slow down a team. If a team member has to ask another team member to clarify something in the ticket, the ticket was not written explicitly enough. The purpose of detailed project tickets is to allow developers to assign work to themselves, complete the ticket, submit their changes to the code base, and grab another ticket to work on.

4. Team members should make it as easy for others on the team to be able to review their contributions to the project.

Team members should be reviewing your code. When a team member does review your code, it should be as easy and painless as possible. Include steps to test your new feature and a list of project files that you changed or added. It is best to be very explicit about what the other team members should be checking when they review your code. Here's a poor example of a message added to a Pull Request for a log in page submission:

> I added a log in page. You can log in now.

A better, more explicit note would look like:

> Users now have the ability to log in and log out. While viewing the home page, a user can see a "Log In" button in the nav bar. The "Log In" button will redirect the user to a log in page. When a user enters valid credentials, the "Log In" button will change to a "Log Out" button. When clicked, the log out button will log a user out.

My team also found it very useful to include the exact terminal commands needed to pull a team member's branch down and get the code running:

```Bash
git pull --prune
git checkout 42-userLogin
dotnet run
```

Code reviews should be painless. If a reviewer has to ask the code author about steps to test his/her submission, the feature author should rewrite the note in his/her Pull Request.

Workflow is extremely important in a team setting. The purpose of working as a team instead of individually is to get more work done. Team members should be able to easily determine what the project goals are, what work has been done, work work needs to be done, and what their next task at hand is.

You can [follow my team's progress](https://github.com/TeamCharles) at Nashville Software School as we continue our group projects through December.
