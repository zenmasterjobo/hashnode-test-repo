---
title: "Enhancing Data Quality Using Better Designed ETLs"
datePublished: Mon Sep 30 2024 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm2ayh5bd00060ale8c8b3yrr
slug: enhancing-data-quality-using-better-designed-etls
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729027742587/c16b7ea7-b52f-4762-a602-10148ce6ed0e.webp

---

# This is a new header

Templates provide a helpful way to produce consistent results across data science teams. They can reduce cognitive load as your peers, stakeholders, and consumers will know what to expect each time. They can also guide you through best practices and can serve as a tool to teach junior team members those best practices and ensure that nothing is missed or forgotten. They can also reduce the risk of something (or many things!) going wrong. This is a new comment that we want in here

At Square, part of a data science role is creating ETLs, which are mostly SQL-based. An ETL design document will ensure that you align your consumers on the technical decisions, as well as explain why they were made. That’s why I am sharing an ETL design doc template that I have created for my team at Square, and below I outline what it may help you with and how it will make it easy to incorporate best practices into your work.

An ETL design doc should also serve as living documentation for your stakeholders and consumers, preventing institutional knowledge from departing with you when you leave the company.

Contributing to data infrastructure and maintaining good documentation is strong evidence of seniority. Using the template will make it easier, as it outlines the steps you need to take to design the template and allows you to explain your reasoning behind the approach you take.

## What will your ETL be used for?

While this may seem like an obvious question, it’s worthwhile to be explicit and clear about it. The Goals section is meant to make you pause and decide: Do you really need to create this dataset? Will it be reused on a regular basis? Whenever you decide to create an ETL, it’s important to outline why you are building it in the first place. For example, it could be because you are re-running a number of queries on a regular basis and want to automate them. Ultimately, there may be a myriad of reasons why you are creating your ETL, and it’s important for you to define them.

The section Key Analytical Questions encourages you to think through what questions this ETL will help or make easier to answer. This is meant to encourage you to truly think about the consumers of the ETL and what their goals are, ensuring that your ETL is both useful and usable to its consumers and that it models the data appropriately for those needs. You can also pair up your ETL design doc with a dashboard design – the latter is explicit about the metrics you will be monitoring.

## ETL design document is helpful for peer review

You should have a teammate (or two!) or a mentor review your design doc. This will be a forcing function to ensure that you have thought through all the possible options – or have someone more experienced make suggestions for a better way to design your data architecture. It’s much easier to review and advise on the design of an ETL than peer review the code you have written and try to decipher what you are trying to do and what the goal is. Alternatively, having a design in place ahead of creation means that you can get feedback on your approach – and change it, if necessary – before building it. This will save you time on iteration.

The detailed documentation on what columns are included, the data types of the columns, and sources of that data will serve as documentation for both your broader team and other teams that may want to use this ETL. Sample values can clarify confusion: will your boolean values be saved as 1 or 0, TRUE/FALSE?

While there are many tools on the market for data lineage and documentation, not all teams or companies may have access to them. Having links to source code and relevant PRDs (product requirements docs) makes it really easy for other people to both look at the code and learn about the broader business context as needed.

## It will help you improve the quality of your work

It’s good to think through different tradeoffs and share what other ways of designing your ETL you have thought through. The Non-goals section allows you to outline what you are deliberately omitting in your ETL and what known gaps exist. If you aren’t including something in your design, this part allows you to explain what it is and why – and that way, your manager or peers won’t think that you have forgotten to add something! It is not uncommon for us to make a decision to exclude something or narrow down the scope of the problem. This allows you to explain this decision.

The Data Quality Checks section ensures that you also include what checks you will do. You should always test your queries and ensure that your logic is sound. It also gives an opportunity for the peer reviewer to make suggestions as to how else you can ensure that you provide a reliable, high quality output. Finally, this should help prevent future bugs or issues that could stem from untested work.

While spending an additional time upfront designing your ETL may feel excessive or even a waste of time, it is a very worthwhile exercise. I have found that it speeds up my overall development process as I don’t need to think through what should be included while writing code and can solely focus on execution. Being explicit on the goals of the template means that I think through and deeply understand the broader business context. For your team, using a template will enhance consistency of everyone’s work.

Special thanks to Will Martin for providing feedback and support for the template.