---
title: "Individual Leadership: The Key to Reviving Scrum"
description: "Revitalize Scrum: individual leadership, micro-sessions, natural feature breakdown, communication, empowerment, accountability"
date: 2023-07-16T05:00:00Z
image: "/images/posts/b7b00f4d-6191-48d4-9b08-bfd3d3c90453.avif"
categories: ["Leadership"]
authors: ["Alin"]
tags: ["Agile", "Scrum", "Software Development"]
draft: false
---
In the early 2000s, Scrum revolutionized the software development industry by replacing outdated waterfall or ad-hoc processes, characterized by long intervals between ideation and delivery, with a more agile approach. By breaking lengthy delivery cycles into smaller, manageable chunks, Scrum facilitated incremental improvements and allowed for *on-the-fly* scope adjustments based on new market insights. Fast forward 20 years, and we have all grown tired of Scrum. It has become ubiquitous and often frustrating, with people forgetting it is the reason the software industry has come this far in the first place.

This article will explore some of Scrum's irritating aspects and suggest ways to overcome them.

### **Too many meetings**

Scrum incorporates numerous *rituals* in each iteration, likely taking up to 10% of the sprint's time. These include daily stand-ups, backlog refinements, sprint demos, retrospectives, and planning sessions (at a minimum). While some of these meetings are necessary, others have their shortcomings. Let's examine the potential issues with some of them:

**The daily standup**:

* **THE PAIN**: You disrupt your focus for at least 15 minutes to listen to what your colleagues are working on and to share your own progress and any obstacles you're facing. If your team is large, you might not be interested or have no clue about what they are saying, as you are likely working on separate features anyway.
    
* **THE FIX**: Create **micro standup sessions** with only the team members working together on a particular feature and then let the scrum master orchestrate the impediments. The SM would need to attend more of these micro sessions (and also some developers working on multiple features) but these should be shorter in length, more focused and engaging.
    

**The planning session**:

* **THE PAIN**: Sometimes planning an item can take more than actually implementing that item which is deeply frustrating. According to standard practice, all team members should vote on the estimation of each feature, providing explanations if the numbers vary significantly. In most cases, only a few work on the feature and they know precisely how long it will take. This Scrum requirement emerged because projects aimed to have as many interchangeable resources as possible, necessitating that all team members stay 'in the loop'. However, in this era where people are either highly specialized or full-stack, this approach no longer seems sensible.
    
* **THE FIX**: As with the daily standup, **create micro backlog refinement** **sessions** with only the people involved in those features, this should be faster and more accurate even if this might be less transparent for the team. You can communicate the findings from these micro meetings to the whole team in the main planning session of the sprint which now should be shorter as everything should be pre-estimated. One caveat: the product owner must ensure that dependencies are managed and essentially prepare the backlog in advance. In the planning meeting, simply present the sprint backlog to the whole team and obtain their approval, or make minor adjustments if necessary.
    

**The demo**:

* **THE PAIN**: The session in which the iteration's output is showcased and receives the PO's approval, which in itself is a great opportunity to bring people together and celebrate their successes but if the team doesn't require such cohesion-building activities (because they achieve it differently), this session can be a waste of time. Moreover, the PO should be already in sync with the developers concerning the acceptance criteria. If they are not, and developers know there is even the slightest possibility of having their feature rejected and work discarded, it indicates a more significant underlying issue.
    
* **THE FIX**: The SM and PO are free to use their time to do the demo alongside the stakeholders - they don't need developers for that. Better still, stakeholders can request demos on demand, just on features they consider important. An even better approach would be for developers to release features (and inform the PO) as they are delivered to the main branch, rather than wait for the demo to present them. This is also supported by the increasing popularity of **developer testing** and cutting-edge CI/CD platforms that facilitate this process.
    

**The retrospective**:

* **THE PAIN:** A lengthy and uneasy meeting where the team discusses what went well during the sprint, but more importantly, what went wrong and how it can be prevented. This presents a fantastic opportunity for improvement; however, the problem is that most of the time, these enhancements never come to fruition - they end up rotting on the sprint's Retrospective board.
    
* **THE FIX:** Let **developers take the lead** on addressing the issues raised during this session - whether it's the items they reported themselves or any other issue of their preference. This can be a **collaborative effort** between one or more developers and the Scrum Master, with the ultimate goal of creating greater accountability and follow-through around these items.
    

### **Too short iterations & forced features**

**THE PAIN**: Most Scrum teams work in iterations of two weeks which considering that 10% of that time is eaten up by meetings, doesn't leave much for development. These short iterations also require you to create small features that can be implemented and tested in this time frame which can lead to nonsensical features just for the sake of making them fit in terms of capacity. It also creates situations where more important features are pushed forward in time as they don't fit the current sprint in terms of capacity.

**THE FIX**: Let the features break themselves up into pieces that **make sense naturally** from both the perspective of software development practices as well as user experience. This may result in a **feature spanning multiple sprints** (ideally no more than two), and that should be acceptable - you will make it part of the sprint goal only when you are confident you can finish it. Consequently, it is expected that some stories will remain unfinished at the end of a sprint and be dragged to the next one. Better still, for products that lend themselves to this kind of features, consider starting with iterations of 3 or 4 weeks.

### **Too much focus on reporting**

**THE PAIN**: In Scrum, reports must be kept updated to ensure correct resource allocation and estimations and manage dependencies. In reality, however, this rarely occurs in a timely fashion and is often influenced by other factors altogether. Resource allocation is challenging, and even if more resources are needed, there is no guarantee that the project will be brought on track. New resources require onboarding from existing developers, making it difficult to predict their short-term contributions to the team.

Dependencies are not easily identified on the spot and are typically discovered unexpectedly or very close to becoming an issue. As a result, reporting becomes a low-thrust mechanism being played by the developers - *similar to counting likes*. They pull in features they know can be completed by the end of the sprint, losing sight of the bigger picture, which is the value they bring to the product.

They even go to such lengths as to 'tweak' the estimates just to make them match the sprint's capacity or cut corners during the implementation so they don't overshoot the sprint. Even worse, they burn themselves out trying to make the feature fit the sprint.

**THE FIX**:  Create your initial team based on rough backlog estimations and adjust the short-term feature scope by removing features if they don't fit but always **focusing on the most important** items. In this manner, the team remains intact while the scope is adjusted to fit - sometimes this can be only temporary (team velocity varies, so in the long term, you might find that you delivered the same scope). If something is indeed too large for the team to handle, then consider outsourcing it to other teams entirely. The purpose of reporting should be to identify teams that are constantly struggling and require external assistance, whether in the form of additional resources, better POs/SMs or a temporary coach to enhance their collaboration/coding skills.

## **The proposed mindset shift**

Scrum was designed as a framework that should work even for the worst organizing teams and if what I am proposing above seems to be best applicable to time-tested teams, I would need to agree and offer instead some new skills and mindsets we should promote among the team members so we can transition faster to less painful software development methodologies:

* **Communication Skills** and **Emotional Intelligence**: To truly enable self-organizing teams, members must be able to communicate clearly and demonstrate empathy. While it is obvious that communication is a crucial skill, reducing the need for "sync" meetings requires individuals to effectively communicate with others without a Scrum Master's intervention.
    
    I am a believer that team members should be **end-to-end problem solvers,** meaning that when they stumble upon an impediment, they should be able to identify who can help them overcome it, initiate communication and follow through until the issue is resolved. This approach not only increases awareness of dependencies but also enhances their ability to communicate these dependencies.
    
    Emotional intelligence is equally important, as it is essential when interacting with new people, potentially involving negotiations around deadlines and resource allocation.
    
* **Empowerment** and **Accountability**: The worst feeling for a team member is to feel micromanaged, as this drains their morale and energy, making them less productive. When people know they are being micromanaged, they don't take risks or assume any responsibility - they simply allow themselves to be led. On the other hand, when we emphasize the importance of taking full responsibility for one's work, **people will naturally step up and lead** - even if, at times, they are only leading themselves.
    
* **Psychological safety**: To ensure that team members can speak up freely, it is crucial to create an environment where they feel safe doing so. Avoiding situations where they might be bullied, ridiculed, or have their words used against them during annual evaluations is essential. **Fostering vulnerability** should be prioritized, and those who act as vulnerability ambassadors should be praised for their efforts in creating a supportive and inclusive work environment.
    
* **Transparency / peer-to-peer organizations**: transparency contributes significantly to creating an open communication environment where people feel comfortable expressing themselves without the fear of retribution. By minimizing the power distance between employees and management, a flatter organization empowers individuals to take on greater responsibility, better work together as equals and also fosters a sense of camaraderie between the team members.
    
* **Team culture**: all these stated above can't work unless you bake them into your team's core values. This is where a team *leader* (not necessarily the official team lead) can help by embodying and nurturing these values. Ideally, these values should not only be shared among teams but also promoted at the company level.
    

## Connecting the dots

All these skills emphasize the importance of **leadership**, not only from team leaders themselves but also at an individual level, reducing the need for rules and processes as team members understand their roles and responsibilities and proactively seek solutions when needed.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><strong>Leadership flourishes through exemplification </strong>so it is essential to have at least one influential individual in every team to disseminate these qualities.</div>
</div>

For small or newly-formed teams, Scrum remains the go-to solution, but cultivating a team culture that emphasizes responsibility, transparency and an end-to-end problem-solving attitude can also lead to effective scaling.

We've misused Scrum, forcing it to work just for the sake of saying we were implementing Scrum but now we're considering abandoning it altogether. Let’s give it one more chance by promoting these mindset shifts, if not for ourselves, let’s do it for the *newcomers* in the field of software development.