# Agentic Development Is the Agile We Were Promised

Tuesday, 6AM. With my coffee in hand I open my laptop to find that Claude had been idle the entire night. It failed to drive Maestro through its MCP to implement the long backlog we stayed up late grooming. The cause is a repeated failure of Claude to survive the Claude 5h window budget reset. So I told Claude how lazy it had been and asked it to continue driving the backlog execution. I hope I did not hurt its feelings.

I come back at lunch time to find that this time Claude has been diligently working on my backlog, and this time it survived the budget reset. That's a big win. It was not easy to get to this point, but things are looking promising.

If you have no idea what I am talking about: I have decided to learn AI. For that I signed up to Claude Max 20x. I started with 5x and soon found that it was too easy to exhaust the weekly limit. Kind of by accident, I started developing an agentic software factory. See [post 1](/2026/06/can-ai-build-an-app-autonomously.html) and [post 2](/2026/07/teaching-the-machine-to-build-itself.html) for more context. This is how Maestro was born. Maestro is a spec-driven, human-steered software factory. Before you ask why I decided to create a new tool instead of using existing ones: reviewing existing tools is on my roadmap. I just decided to start by creating what I think would be my ideal agentic harness, and then compare with existing options.

## The core problem: "done" wasn't done

My initial goal with Maestro was to have no contact with the code — a tool that product folks could use. You still need to know about software development, but I wanted to remove the need to read and approve every single line of code. I have been mostly successful in that, but it took a few stops to change some fundamentals of how Maestro works.

The main problem that I repeatedly found was that Maestro would tell me something was done, and later I would find it was not really done. Or it was done, but it also included a few extra things it decided along the way. Or worse, it removed functionality without asking.

Before I list what we have done to fix these problems, here is a quick overview of how the work flows:

Requirement is filed → optionally a prototype is generated → approve → a plan is generated → approve → the plan is executed → if successful, it goes in the next deploy → I manually deploy every few requirements.

## The retro with Fable 5

At this point I decided to do a retro with Fable 5, and we came up with a few really good suggestions.

1. **Feed the architecture into the planner.** That was a gap. It should have been like this from the beginning.
2. **Invariants.** Now there are certain things that are baked into the architecture that must be respected.
3. **Self-analysis on failure.** Once a plan fails to complete, Maestro introspects to find the cause and suggest next steps. Usually it involves breaking down the requirements or replanning.
4. **Lessons.** The self-analysis also proposes lessons that, if approved, are used in the planning of following requirements.
5. **Automation.** The first task of every plan is to create a test specification that starts red and must pass at the end of implementing all tasks of the plan.
6. **Prototypes fed to planning.** Another one that should have been like this from the beginning.
7. **Manual approval.** The last, and probably most important change: a successful plan execution now needs a manual approval.

Everything until step 6 made Maestro better, but the manual approval is the one that is proving to give the best results. This approval is done only based on the working software, linking to the initial idea of product folks being able to use it. If rejected the requirement is sent back to the kitchen, and Maestro starts again with an optional prototype, plan, and execution. I can approve or reject again.

## What fast agile actually feels like

Now, almost 2 months into this journey, I finally feel Maestro is really becoming an agentic, human-steered software factory. I've been dogfooding Maestro's requirements into itself, so it is essentially building itself. Besides Maestro, I used it to do a few POCs. Now it's time to test it with other real products.

This experiment with building Maestro made me see what agile would look like if done properly. The feedback loop is so fast. I manifest something and it is done — but not perfectly. I see the result and ask for another increment in the direction I want it to go, but with real feedback to make a decision. This is how I see the future of product development unfolding.

This approach will not resonate well with many developers. I would not have delivered something I built like this before a thorough inspection. Maybe this is where the more capable models, like Fable 5, come in. Seeing a complex product working without me having to write the code makes me feel powerful, and there is no denying that this can at least give really fast feedback to product development. Let me see where this goes next, and I will come back to tell you. I still have to find a way to make Maestro stop duplicating code and add support for different providers. This is where I am going next.
