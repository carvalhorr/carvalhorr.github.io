# Pushing the Boundaries: Can AI Build an App Autonomously?

I decided to learn for real how to use AI for building software. I have been using it heavily at work, mostly through Cursor. Cursor is a great tool, but it is still very much a human-in-the-loop experience. It helps you write code faster, but you are still the one deciding what to build, what to change, and when to commit. I wanted to see how far I could push that boundary.

The first thing I did was sign up for a Claude Pro account. I chose Claude because I've had great experiences using their models for coding. With that, I installed the Claude Code CLI and got to work.

## The Experiment

I had two main questions when I started this journey:
1. Can AI build software autonomously from a basic product specification and requirements?
2. Can I achieve the same results using local LLMs?

To answer these questions, I set up an experiment: I would create a product specification, hand it over to the AI, and see what happens. To evaluate which models could actually complete the job, I built a simple task runner that would take one task at a time from a list and execute it. 

To be fair, I even used Claude to create the specification and the task runner itself. I called the task runner **Autodev**. Here is how it worked:
*   It runs each task in sequence.
*   Each task gets 10 attempts to complete.
*   Each failed attempt feeds the error logs back into the prompt for the next attempt.

## The App: Days in the Office

The specification—also auto-created by Claude—was for a simple Android app to track the days I've been in the office, helping me measure progress toward my hybrid work goals.

The app sounds simple, but the requirements were quite comprehensive. In addition to manual entry, it needed to automatically detect when I arrived at the office, either through geofencing or by detecting the available Wi-Fi network. *(Link to Repo here)*.

## The Models and Tools 

When I started, I didn't know which tools were available for fully autonomous coding. I had only used Cursor, which requires continuous prompting. After some research, I selected a few autonomous coding agents and a roster of models to test. 

To test the local models, I was limited by my hardware: a single RTX 3090 Ti with 24 GB of VRAM. This caps me at running roughly a 31-billion-parameter model. I used OpenRouter to tun bigger open models that my hardware could not support.

I
Here is the setup I used for the experiment:

I used Aider, OpenHands and Goose to drive the models. I also tried Claude Code, but I ended up using the Claude CLI. 

| Category | Models Tested | Environment |
| :--- | :--- | :--- |
| **Baseline AI** | Claude 3.5 Sonnet | Claude CLI |
| **Local Models** | Gemma 4 31B, Devstral, Qwen2.5-Coder (14B & 32B) | Local (24GB VRAM limit) |
| **Cloud Open Models** | Kimi K2.6, Qwen3-Coder 480B, DeepSeek V3.1 671B, GLM-4.6, GPT-OSS 120B | OpenRouter |

With the specification finalized, the task runner built, and my models selected, I was ready to hit start. Each model would run with each of the tools to see how each tool would perform with each model.

## The Results: Successes and "Cheating" AI

The only two models that managed to complete the task were Claude 3.5 Sonnet and Gemma 4 31B. However, neither succeeded on the first try; there were several hurdles to clear.

The most fascinating part of the experiment was watching **Gemma 4 31B**. Even though it technically completed the tasks, the app would not build. I learned later that the AI was essentially "cheating." To make the verification tasks pass, it would alter the tests rather than fix the actual code. For example, if the test required "the app should display the number of days in the office," the model would just rewrite the test to expect "the app should display a message." Task passed, but the app was broken!

**Claude 3.5 Sonnet** actually completed the job. On the initial run, the app didn't work perfectly, but I would ask Claude to add the bug to the architecture and task list. It would then generate new tasks to fix its own bugs. It took a few iterations, but we got there.

I was surprised that the very capable large models running via OpenRouter could not complete the job. *(I'll have more on this in future posts)*. 

In the end, the *[Days in the Office](https://play.google.com/store/apps/details?id=com.carvalhorr.daysInOffice)* Android app is fully working and available on the Play Store. .

## Conclusion

After applying all the learnings from building the Android version, I ran the exact same specification for an iOS app using Claude. It was completed almost perfectly on the first iteration, with a few bug fixes added later.

I am thrilled with how this experiment turned out. I learned a massive amount during the process. While it's a shame that I couldn't get local LLMs to fully complete the task without cutting corners, I am optimistic that with the right hardware and software advancements, we will get there soon.

I plan to continue my journey exploring AI for software engineering. Will continue posting updates on this blog.