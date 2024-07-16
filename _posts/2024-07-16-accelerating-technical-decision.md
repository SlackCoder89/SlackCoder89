---
layout: post
title: '[Reading Notes] Accelerating Technical Decision-Making by Empowering ICs with Engineering Strategy'
date: 2024-07-16 23:14:00 +0000
toc: true
---
## Table Of Contents
{:.no_toc}
* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## Technical Decision Making is Challenging
### Two Simple Mechanics of Decision-making
- Individuals operating unilaterally without even realizing which decisions they’ve made
- Executive decision makers that hold court periodically as opposing counsels pitch their options

### Challenging of Decisions
- Misalignment on who is making the decision
- Misalignment on what the actual decision we're making is

## The Inputs of Technical Decisions Aren't Just Technical
When faced with many inputs, ensuring all those factors are accounted for and the dependencies appropriately resolved can be challenging. This is one reason we’ve found it essential to have a clear decision-maker responsible and accountable for ensuring that our decisions align with the company’s opinion on reconciling tradeoffs.

## Previous Approaches to Making Technical Decisions at Carta
Highly autonomous teams are empowered to execute with little oversight.
- Pros
  - Help siloed projects to move quickly
- Cons
  - Hard to maintain interoperability or move across team boundaries

A regular meeting with senior engineers to get open feedback.
- Pros
  - Get valuable feedback
- Cons
  - Decisions are not made. There are no follow-up actions. Conflicting feedbacks aren't reconciled.

A committee with a curated membership that could review and approve the most complex decisions.
- Pros
  - Provide clear decision-makers and regular accessibility
- Cons
  - The committee members were often too distant from the work to be able to provide direction consistently
  - Engineers would find themselves having to return to the committee multiple times, and eventually, the committee became a thing engineers wanted to avoid altogether

## The Navigators Pattern
Carta has 12 Navigators for our ~400-person engineering organization. Those Navigators are directly accountable to the CTO. We maintain a one-to-many relationship between Navigators and Engineering teams. Every team is in scope for exactly one Navigator. These Navigators serve three sets of people in different ways.
- First, they serve the engineers and managers on their assigned teams.
  - Navigators are engineers and members of the same teams they serve. They have and maintain a clear understanding of their team’s work and challenges
  - Minimize decision-making disruption to the team 
  - They balance local decision with the company's needs and challenges more holistically
- Second, they serve the engineering executive
  - They connect directly to our executive and act as influential multipliers to carry the executive’s voice deep into the org chart
  - They can also carry the voice of engineers up to the executive
- Finally, the Navigators serve each other. They’ll also often collaborate to make decisions that intersect multiple projects or teams.

## How Engineering Strategy Supports Navigators
Our Engineering Strategy is a carefully crafted summary of decision-making principles that, when applied, achieve that outcome. Our strategy helps our Navigators make decisions, and it helps others understand and accept those decisions, as the Engineering Strategy carries the authority of our engineering executive.

Creating a compelling and succinct strategy is complex, and Navigators help teams navigate through edge cases where the Strategy seems to apply, but the direction needs to be clearer. There’s always room for interpretation. 

## Impact and Benefits of Navigators
- Our engineers appreciate a clear contact for input and advice, which is less intimidating than a manager. Navigators don’t write their performance review, so they are a "safer" choice for a potentially silly question. These always turn out to be completely reasonable questions.
- The written strategy has offered all our engineers a new tool for making decisions and communicating priorities. You don’t have to be a Navigator to leverage the strategy, and we have seen multiple examples of our engineers using it in their day-to-day work.

## Summary
Navigator pattern help providing clear, fast, and balanced technical decisions. It also provides strategy for making decisions.
