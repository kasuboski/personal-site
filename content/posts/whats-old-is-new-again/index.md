---
title: "What's old is new again"
date: 2026-11-15T00:00:00Z
draft: false
---

Coding agents have given me more reasons to invest in the developer tooling I've always cared about.

<!--more-->

I've always been interested in platform engineering and making it easier for other developers to get things done. Working with coding agents has given me another reason to invest in planning, documentation, development environments, and useful checks.

A lot of this was already a good idea. I just couldn't always justify the effort alongside everything else that needed doing. Agents can help build the tooling, then use it to check their implementation before I review it.

[Devon Estes wrote about reconsidering ideas](https://devonestes.com/bad-ideas-that-arent-bad-anymore) whose costs have changed with AI. I like that framing. Some practices deserve another look because they're cheaper now. Others become more valuable when agents produce more code than I can review in detail.

## Planning with agents

Agents can help with planning too. An agent can investigate the existing code and compare approaches. If we're unsure about an assumption, it can build an experiment to test it. Trying a couple of implementations can now be a reasonable part of planning.

I still need to decide what problem I'm solving and which tradeoffs I care about. But I can get much further into understanding those tradeoffs before committing to an approach.

Deciding which module owns a behaviour and what it can depend on gives the agent constraints to work within. I'd rather establish those before reviewing a working implementation that doesn't fit the application.

I've heard this compared to running a kitchen, although I don't remember where. The developer is the head chef and delegates work while remaining responsible for what reaches the customer. I like the analogy because the chef establishes how the kitchen works, including who handles each task and how to check the food before serving it.

The same applies to a development environment. If I want to delegate more work, it's worth making the expectations clear and giving the agent a way to check what it's doing.

## Keeping the reasoning

Some of those expectations come from decisions we've already made. The code doesn't necessarily explain why a feature exists, why a seemingly unnecessary restriction is there, or why we decided against the more obvious implementation.

We've always had places to write this down: design documents, commit messages, PR descriptions, and comments. Now a lot of that discussion happens in an agent session. If we only save the resulting diff, we lose the discussion behind it.

I'm watching [Entire](https://entire.com/) and [Delta](https://delta.dev/) because both connect the conversations behind changes with the code itself. Being able to recover the discussion that led to a decision seems much more useful than asking the next agent to guess why the code looks like that.

I still want the final decision to be easy to find. A long conversation might include abandoned approaches and assumptions we later discovered were wrong. I'd want a summary of what we decided, with the conversation available if I need the details.

That gives the next agent context to work from. For some decisions, we can go further and give it a check that tells it when it's gone against them.

## Making the expectations executable

Human reviewers shouldn't need to mention formatting in a PR. We have formatters, CI, and a nice UI showing which checks passed. The author should have that feedback before another person spends time reviewing the change.

The same setup works well for agents. They can run the checks, read the failures, and make corrections before asking for review. I'm increasingly applying this to anything we can check automatically.

I've been turning on stricter linters in my projects, including Clippy's pedantic checks for Rust and treating Glinter warnings as errors in Gleam. This creates more things to fix, but an agent can work through the feedback as part of implementation.

I still need to choose which rules apply to the project. Once I've chosen them, a failing check can remind the agent without me repeating the instruction in each session.

The same applies to custom checks. If one module shouldn't depend on another, that's a candidate for an executable rule. Previously I might have left it as a convention because writing the check felt like a distraction. Now I can ask the agent to implement the check as part of the work.

Agents can use these checks to verify their progress. They make a change, run a check, inspect the failure, and try again. That lets them correct mistakes without waiting for me to point them out.

This also makes review more useful. The routine checks have already run, and I can spend more time on whether the change does what I wanted and fits the application.

A passing test only verifies what it asserts. If the assertion doesn't match the behaviour I want, the agent can pass the test and still produce the wrong result.

## Actually running it

I want the agent to run the application and exercise the workflow it's changing. Running just the affected part is fine if it includes the dependencies needed to test that behaviour.

I've been adding full browser tests for the main workflow. They capture screenshots for a gallery I can review on PRs alongside the CI results.

That makes a change much easier to review than a description saying the UI has been updated successfully. Assertions can check that the expected elements and behaviour are there. The gallery gives me a practical way to inspect the result without manually reconstructing every state.

Getting the application into those states can be a fair amount of work, especially when it depends on external services. Simulators have been useful here.

Building a fake version of a dependency gives the application something predictable to develop and test against. This is especially useful for LLM calls, where I don't want every test to depend on a live model returning a convenient answer.

A simulator can return a known response, introduce a delay, or fail in a particular way. That lets the agent exercise the surrounding workflow repeatedly, including the awkward cases.

I'm adding more end-to-end tests with these simulators. Flaky tests and difficult setup have always made me cautious about adding more of them. Those problems still exist, but agents can help diagnose failures and fix the test setup. That makes the maintenance easier to justify.

I don't mean rerunning a test until it happens to pass. I'd like to know why it failed.

The simulator needs to reproduce the relevant behaviour of the real service. If it always returns a valid response, the tests won't tell me whether the application handles an invalid one.

The [AWS Emulator write-up](https://www.aboutamazon.com/news/aws/how-autonomous-ai-agents-built-aws-emulator) is useful here. Their agents develop tests against real AWS and use those tests to evaluate the emulator. They have a reference for the behaviour they're trying to reproduce.

[T3 Code's verification tooling](https://github.com/pingdotgg/t3code/blob/main/docs/operations/background-verification.md) is another example I like. Its background-work checks run the real server and provider integrations. They verify that the agent retrieves and acknowledges a result, rather than just checking that it was delivered. The documentation also says which behaviours need separate tests. That's the level of detail I'd want before trusting a passing check.

## Improving the checks

I also want to check whether the tests catch mistakes in the code.

I've added mutation testing to help with this. It deliberately changes the code and checks whether the tests notice. Code coverage can tell me a line ran. I'm more interested in whether changing its behaviour would cause a test to fail.

If mutation testing reverses a condition and every test still passes, I want to know why. I don't want another score to chase, but I can ask an agent to investigate the surviving mutation and whether we're missing an assertion.

The agent session is another source of feedback about what's missing.

There's a detailed record of what it tried, which commands failed, and where I had to intervene. It isn't complete access to the model's thinking, but it's a useful record of how the work happened.

If I had to explain an architectural constraint, could that become a check? If the agent struggled to reproduce a problem, could we add a fixture or a command that makes it straightforward? If it stopped after unit tests, could the relevant browser test become part of the completion criteria?

The session can show where we need documentation or better tools. It can also show that I hadn't decided what I wanted. I need to resolve that before asking the agent to write a check for it.

Saving the session lets me revisit those interventions. Adding a check or a reproducible command means the next agent can detect the same problem before asking me to review its work.

## After it ships

These checks give me evidence before shipping, but production can still expose failures the tests missed.

I need enough observability to understand what changed and whether the feature is behaving as intended. I also need a practical way to respond if it isn't, whether that's disabling the feature, rolling back, or fixing forward. Data changes can make that decision a little less convenient than redeploying the previous image.

"It worked when I tested it" has never been enough if the change causes problems in production. "Claude told me this would be great" isn't much of an improvement.

Delegating implementation still leaves me responsible for understanding the evidence, deciding to ship, and dealing with what happens next. The checks and tooling help me do that, but I also need to understand where their coverage ends.

I'm still finding more things to automate so agents can produce what I want before I review their work. When I have to correct something, I want to check whether the project could have caught it earlier. That gives me a practical way to decide which tooling to build next.
