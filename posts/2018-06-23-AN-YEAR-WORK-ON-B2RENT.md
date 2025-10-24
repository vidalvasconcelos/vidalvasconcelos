# A Summary of a Year with B2rent

- Date: 2018, Jul 23

About a year and many cups of coffee ago, I was invited to work on a project in 
Fortaleza. That was what finally made me move to a new city—something I had 
been postponing for two years, insisting on staying in Sobral.

The project was incredible: working on a Laravel 5.1 application aimed at helping 
companies manage rentals. Initially, I was going to be the sole developer leading 
this endeavor, with B2rent being just one of the products of Pilps GP.

I felt like the perfect developer for the job. Working with legacy code in a 
startup environment had become almost second nature over the past two years. That 
period—working as a freelancer or building my own startup—was one of the best 
times of my life up to that point. I met many people who helped me and radically 
changed my worldview. 

That energy of creating new things and learning every second is an intrinsic 
part of who I am today. The challenge was to expand the project's functionalities 
while streamlining the codebase—a Herculean task, to say the least. 

A prime example of this is that, even a year later, there are still Controllers 
I have never touched. The reason for the bloated codebase was the number of 
developers who had worked on it before me, implementing isolated features 
without considering maintainability—each with a different vision of what the 
project should be.

Most tasks always came with extensive refactoring of the codebase: removing 
business logic from Controllers, optimizing massive queries, extracting 
useless code, and, more often than not, negotiating scope adjustments. Through 
this, I learned that much of what makes software complex is the lack of 
structured processes on the client’s end. 

The result of this work can be seen in the following graph, which shows the 
number of lines added and removed in my contributions to B2rent. These numbers 
aren’t what truly matter—the key takeaway is that, even as the product gained 
more features, we managed to keep the codebase as lean as possible.

| Commits | Lines Added | Lines Removed |
|---------|-------------|---------------|
| 404     | 603,784     | 1,875,203     |

The frontend was another challenge. Laravel comes with a built-in structure for 
Vue.js 1.0, but that setup had been completely removed at the project's start. 

Since I was particularly a fan of the library, one of my first tasks was to recreate 
its setup. Although Vue 2.0 had already been announced at the time, bringing some 
incompatibilities, we built most of our components with the new version’s requirements 
in mind.

The goal was to reduce the percentage of jQuery and plain HTML while increasing 
Vue’s presence by creating components of various parts of the application. We also aimed 
to remove AdminLTE from our app. I have nothing against templates like that, but 
if you don’t want to give the impression that you’re just another generic app, 
don’t use them. 

Another lesson I learned is that, to scale, you need a simple, agile, and flexible 
structure. This has made me rethink the use of frameworks, but that’s a topic for 
another post. We evolved from a bloated, static admin template to a lean and 
efficient app.

This wasn’t just my work alone, but I’m proud to have led this project, worked on 
its identity, and helped design every feature we built. This post may feel like a 
farewell. After a year of living and breathing this project, so much has happened. 

Our team has grown, and other projects within Pilps have also expanded, demanding 
more of my time. Currently, I’m working full-time with TypeScript, and this shift 
has motivated me to write more. In a future post, I want to share how we migrated 
our frontend to Vue.js 2 + TypeScript and how this transition helped balance 
knowledge across all our projects.

