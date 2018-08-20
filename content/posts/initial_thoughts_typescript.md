---
title: "Initial thoughts on TypeScript"
date: 2018-06-23T14:09:43-08:00
draft: false 
---

I am unsure how I feel about TypeScript in general. The promise that it is *just js* does not resonate well with me. Have a legacy project with amd style js and a lot of custom code? Good luck switching over quickly... The idea of just renaming to .ts does not work in the real world, at least not for me. I believe it is possible I just do not have the time to dig through. The other issue I have it that everything needs a definition file. Most/all are created by the community and this, as always, leads to a bit of churn or incomplete definition files. 

With a few of my issues out of the way, the part that has me sold is the ability to catch bugs before going live. JavaScript can use a bit of this. I love the flexibility but I also hate getting stung because I did not catch everything that could be considered false or forgetting to return the correct "type". What really sold me was how many great projects are authoring in TypeScript and Flow. If the best in the business are going this route it is worth looking into. 

The other reason I am giving it an honest try is that the company I work for is stacked with C# developers that "just get by" in JavaScript. They try to write JS like C# and do not want to understand the idosyncrisies of scope and closure in JS. TypeScript allows this team to become more powerful with the option to opt out. You also get all of the latest goodies and some nice IDE integrations. 

So, in order to give it a go I thought I would write a new piece of a project in Nuxt.js with TypeScript. I used the template found [here](https://github.com/nuxt-community/typescript-template) The template is great and has a bunch of things ready out of the box. The only question that I am left to figure out is the testing story (future post). After getting set up and migrating slowly into the TypeScript waters I think I am sold. At least initially. I will follow up with some of the gotchas I find. Again, the only reason I am able to do this is I can get going with a brand new setup for this portion. Even if I love it I do not have the time to convert anything else... Still worth it? 
