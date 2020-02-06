---
layout: post
title: The ART - or RAT - of Testing
categories: testing
tags: [testing]
author: AnitaLipsky 
---

# Introduction

Something that came up as part of a shared work session in a previous QA
team I was part of is the acronym ART — or RAT, ha ha! still makes me smile —
that we used when when designing and reviewing tests.

In short, we realised the following were the the indicators we always looked for
when working on tests.

- **A**sserts: Do the tests assert the right things?
- **R**eadability: Are the tests easy to read by anyone who might need to
update or refer to the tests?
- **T**ested: Are the tests covering scenarios that are valuable?



# Asserts

For Asserts, this means asserts in the tests should be there and assert the right
things. Tests can have one or multiple asserts. Without asserts the tests have
little value, since they can pass even if the underlying functionality is broken.

Keep an eye open for test code that potentially could result in false positives or
false negatives. False positives are false alarms: the functionality is working
correctly but the test fails, which is typical of brittle tests. False negatives are
when the functionality does not work correctly but the test still passes, thus
missing nasty bugs, which is typical when asserts are missing or asserting the
wrong things.

In all honesty I regularly saw asserts simply forgotten, which is normal when
team members are under time pressure and such. No judgements on this, only
quick and neutral feedback, I promise :)

True story: in a job interview I was asked what I looked for when code reviewing
tests, and I turned to my trusty ART — or RAT — of testing to answer. However,
I completely blanked on the A. In other words, that sneaky Assert was
forgotten, even in the meta sense! So again, no judgements.



# Readability

For Readability, this means the coding style needs to be simple and obvious
enough to the key players who will be involved in writing, maintaining and 
reading the tests. No fancy coding styles allowed if key players or future key
players might not be familiar with them. Be realistic - there is a limited pool of
resources, and don’t assume they all have a taste for the latest syntactical
sugar.

Readability also means the human language used in the tests needs to be clear
for all stakeholders. For example, when writing tests to match user stories (As
a… I would like to… In order to…) be sure to fill in the blanks with phrases using
words that describe the features as the clients would, to ensure everyone
understands what that feature is. In other words, no cryptic shorthand or weird
expressions only some developers might understand: clear, transparent
language actual humans would say to each other it important.

At the end of the day, software is made by humans — don’t talk to me about “AI
takeover” or recite “2001: A Space Odyssey” robot rules to me — so let’s use
human language wherever we can.



# Tested

For Tested, this can be a bit tricky, because how do you know when the test
coverage is enough?

Many questions will arise here because testing is hard, as software is complex,
and lots of things will and do go wrong.

Though, testing is also wildly fun, because it really can bring a team together
and bring out the best in everyone — apart from those moments where it does
not do quite that! :)

Test coverage comes down to using a mix of experience, theory, and a bit of
luck. For experience, learn from your team, both business facing and tech
facing, ask all the “stupid” questions, and try to cover the most used scenarios
and complexities first since it will usually be a race against time to get test
coverage. For theory, the International Software Testing Qualifications Board
([https://www.istqb.org](https://www.istqb.org/)) Foundation Level Syllabus is really handy here because
it walks through different test types that can be used for different situations.
For example, it explains when to use Equivalence Partitioning, Boundary Value
Analysis or Decision Table Testing, among many other things. The luck part
comes in because typically there is never enough time to test everything due to
the complexity and thus amount of test coverage needed, so pick your battles,
and keep moving to the next most valuable area to add test coverage.

Ultimately, look for evidence tests will fail when important scenarios will stop
working. Even better, run the tests with the test environment tweaked to fail for
that scenario, and look for test failures.



# Conclusion

Making software, testing, and delivering software of value is hard — although
lots of fun!! — so I encourage simplifying focus by using the ART, or RAT, of
testing. In this way, your tests should be more readable, easily maintainable,
and test what matters most.
