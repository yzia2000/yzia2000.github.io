---
layout: post
title:  "Yousuf's Feynman Technique"
date:   2021-01-21 18:43:48 +0530
categories: Blog
---

My workload has increased tremendously this semester, which might be obvious
since I am only a couple of semesters shy of graduation. One half of my day is
spent learning concepts and the rest of the day is spent implementing them. This
drastic change motivated me to look at ways to improve my cognition which made
me chance upon a familiar topic, the Feynman Technique. The Feynman Technique
draws a connection between the understanding of a concept and explaining it
and I found its implications quite interesting and relevant not only to fellow
students but also experts and professionals.

## Overview
Richard Feynman was a theoretical physicist, popular for his work on subatomic
particles, quantum computing and the controversial atomic bomb during World
War II. A huge part of his popularity was his ability to explain concepts to
individuals not thoroughly knowledgeable in the disciplines he professed which
led him to implement a mental model after it, the Feynman technique.

According to the technique, in order to refine your understanding of a concept,
you must first explain the concept to someone as clueless as a toddler, identify
the gaps in your explanation, review and simplify.

<div align="center">
<img src="{{ "/assets/images/feynman.jpg" | relative_url }}" />
</div>

This technique makes sense not only from a psychological standpoint - you
most certainly don't want to disappoint a toddler - but also from a theoretical
perspective. Explaining to a toddler might imply breaking high level knowledge
to low level chunks of knowledge that can actually be understood. This forces
me to believe that maybe true understanding of knowledge comes from understanding
of its derivation from the lower level constituents.

## Graphical Implication of the Feynman Technique
In order to understand idioms, you most certainly need to understand words and
in order to understand words you need to understand alphabets. Does this sound
familiar? This makes me visualize knowledge in the form of a graph. Moreover,
to prevent circular reasoning, we can assume this graph is acyclic. 

Knowledge seems to flow like a topological sort from its lower level chunks
to higher level chunks.

<div align="center">
<img src="{{ "/assets/images/toposort.png" | relative_url }}" />
</div>

In the above image, if each node represents a concept that builds up with
an outgoing vertex, a topological sort of the graph makes us realize that
knowledge will flow in a sound manner from 7 to 0. Topological sorting helps
solve problems like project build dependencies. Maybe it can help discern
dependencies in the cognitive domain.

One might find it ridiculous but with research backing the positive correlation
of mind maps with cognitive understanding, this theory is not too far from
actual implementations.

If we can traverse this graph smoothly both top-down and bottom-up, we can certainly
explain the target concept to most individuals and that in turn has a transitive
effect in proving that we have achieved true understanding. However, this knowledge
theory clearly comes with drawbacks.

## Drawbacks

1. It's time consuming. Sometimes, all you need are the higher level details.
2. Humans are very good at GIGO (Garbage In Garbage Out). One can believe a
perfectly unreasonable theory and preach it too.
3. It is susceptible to false negatives. Sometimes, the fault doesn't lie in
our understanding but the mental faculties of the other person.
4. Communication delay.

## Refining the Feynman Technique
Majority of the problems lie in human interfacing and other confounding factors
like time. However, if we view the theory simply from my perceived implication
of the technique, a somewhat modified proposal can be made.

This refinement breaks learning into the following steps:
1. Identify knowledge graph.
2. Toposort the graph.
3. Traverse the resulting array.
4. If stuck, refine the knowledge graph.

## Concluding Remarks
After going over this journey of trying to visualize cognitive function, I realized
that viewing knowledge as an acyclic dependency graph can perhaps provide a
different perspective to grasping difficult concepts. 

In hindsight, my implementation of the of the Feynman technique might defer
both psychologically and theoretically from yours. Moreover, some assumptions
like the acyclic nature of true knowledge might not stand in all scenarios.
Sometimes you have feedback loops. However, if you found it useful or have
suggestions do post it on the [discussions forum](https://github.com/yzia2000/blog/discussions).
