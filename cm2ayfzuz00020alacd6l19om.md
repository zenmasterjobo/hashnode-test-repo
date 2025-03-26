---
title: "Peer Reviews for Data Science"
datePublished: Mon Sep 30 2024 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm2ayfzuz00020alacd6l19om
slug: peer-reviews-for-data-science
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729027689110/5127dc10-808d-4928-9cf6-3c7007be603c.webp

---


Peer reviews increase the quality of data science output by diversifying our thinking and decreasing the probability of making an error.

Once you’ve felt the terrible sinking feeling in your stomach, you never forget it. I was in my first year as a data scientist in my first industry job, still getting used to the tools and cadence of the tech world. My assignment was to analyze a fairly simple A/B test, but one with company-level revenue implications, and I had just completed the read-out to the engineering team. I returned to my desk to answer some follow-up questions, and was scanning my analysis code from the top to figure out what parts needed to change when I noticed the inner join that should have been a left join. I had unintentionally excluded a bunch of zeros from my analysis. Everything I’d just spent an hour explaining to a dozen very interested engineers might be wrong.

At that point in my career, I wasn’t in a particularly unusual position — today’s data scientists, even relatively junior ones, have a broad scope and a large degree of autonomy. While these circumstances are great for engendering the creativity and ownership necessary to produce high-quality data science output, they can also result in excessive specialization to the detriment of the team, and (as I had just discovered) errors going unnoticed. I took a deep breath, fixed my query, and re-ran my script. As I waited, I spiraled, wondering how I could have missed the error.

Through nothing other than dumb luck, the error turned out not to affect my overall conclusions. I updated my document, added a note on the correction, and started following up on the questions. Nevertheless, that moment of terrible uncertainty left its mark, and when I started leading other data scientists I wanted them to learn from my mistake. In that case, I found my own error through repetition and luck, but at Square, we’ve found that peer reviews are more effective at catching issues than hoping we’ll all notice our own. They also help break down barriers between data scientists, forge a stronger team community based on shared knowledge, and ultimately increase the quality of data science output.

So what are data science peer reviews? The process we created at Square is inspired by two related traditions: code reviews in software engineering and peer reviews in scientific research (I spent 10 years as a neuroscientist in the days before widespread adoption of bioRxiv enabled broad dissemination of unreviewed work). There are many great discussions of those processes in the public literature, so we won’t spend any time discussing them here. Data science work requires tools from both endeavors, so we have attempted to define a review process appropriate for this newer discipline. The goal is to gain the advantages of having more than one person checking for errors, looking for missing pieces, and thinking about problems and solutions. A secondary benefit of peer reviews is that they spread information and context across the team, which reduces silos and creates more shared understanding and team cohesion.

At minimum, there are two people involved in data science peer reviews — the primary producer of the work and the reviewer — and we’ll discuss expectations for each.

## Expectations of the producer
The primary producer of any data science work is ultimately responsible for the quality of the output. Without vigilance, teams will tend toward the producer being left alone with this responsibility and not given adequate support and structure around improving their output. With peer reviews, producers are also responsible for identifying one or more reviewers when they start work (roughly corresponding to when a ticket goes from to-do to in-progress, if you track your work in tickets). Producers should share their code, written documents, figures, etc. in a format that makes in-line commenting possible. Finally, the producer is responsible for letting the reviewer know when the work is ready to be reviewed, sharing links to relevant context and related work, and providing a few representative examples or test cases that they used to manually verify their approach.

## Expectations of the peer reviewer
A peer review should consist of the following three parts:

- Direct inspection (and execution!) of the code
- Spot-checking several examples
- Checking against alternative data sources or methods and existing analyses

For very long analyses, review the logic and technical details for the key results first. This is one of the clear time-saving benefits that makes the review take much less time than the production — the reviewer has the benefit of hindsight, and only needs to check the assumptions and findings that turned out to matter. Given the ever-present need to prioritize our time, we recommend spending less time checking less important parts of the work.

