Delegating Work to GitHub Copilot in 2026
Interactive agent usage in GitHub Copilot is so 2025! Learn the new ways of delegating a whole task to a remote coding agent and boost your productivity!
Executive Summary
This article provides simple instructions for delegating work to Copilot, enabling remote development without developer intervention. I believe this is a key enabler of big productivity improvements in 2026. The new agent orchestration view might be the future IDE for developers.

Call to Action
Migrate your source code to GitHub and onboard to GitHub Copilot. This is a one-stop shop for a broad range of facilities, be it the actions system, agentic development locally, or in the cloud. AI can help create CI/CD pipelines from e.g. GitLab to GitHub. Contact Visma IT to help with migrating the data and PRs, as well as get Copilot cheaper under the Visma MS enterprise umbrella.
Can you run your product on Linux? If not, consider modernizing to avoid license costs and improve performance. Our modernization workshop can help here.
Make sure your developers don’t run out of premium requests. The investment is very low (tens to hundreds of dollars per month), and the productivity gains can be measured in thousands or even tens of thousands. This also benefits their upskilling.
Follow up with your team on how much AI delegation is being used
Where does the background coding agent work, and where doesn’t it? When it doesn’t, instead of saying “AI doesn’t work”, ask “How can we instruct it better?”
Likely, an answer will include having better documentation of your system in the repository itself. Then you can reference that documentation in the task, or let Copilot find it automatically using its RAG database.
I collect monthly stats on the percentage of AI-delegated work and AI code review. Unfortunately, it can’t track all AI agents since they masquerade as if the PR was created by the developer.
What is This?
Read GitHub Copilot Coding Agent: An introduction to your new teammate for details on the difference between interactive agent mode and the remote coding agent.

Delegating Work
There are multiple ways to delegate work to Copilot for background execution. These require your source code to be on GitHub.

Assign a GitHub Issue to Copilot on the GitHub Website
GitHub issues are a lightweight issue tracking system. Most of us use something else, like Visma Jira. However, if you have an issue in GitHub, you can easily assign it to Copilot.



How to create such an issue, especially if you already have the task described in Jira? Fear not! This can be automated. Read The Devin Killer? Async Fix a Jira Issue Using GitHub Copilot, which uses MCP. Then, you can just say “/async_fix_jira_issue ABC-123” which fetches the Jira issue, creates a similar GitHub issue, and assigns Copilot to it.

You can see the progress of Copilot by clicking on “View Session” in the draft Pull Request:



Via the Agents website
The website at https://github.com/copilot/agents allows you to describe a task and assign it to Copilot.

Via the IDE
You can describe your task in Visual Studio Code, and delegate to the background coding agent via a button.



Note that you can also delegate to the background. This runs on your local computer in a separate git worktree repository.

Environment Configuration
The environment for the Copilot Coding Agent can be configured in the magic file .github/workflows/copilot-setup-steps.yml. Read up on it. Note especially that environment variables and secrets are managed differently from the ordinary GitHub Actions system. The environment only supports Ubuntu; unfortunately, no Windows environment is available.

Plan Locally, Delegate to Cloud
A nice workflow is to use the Plan mode to plan your changes, and then use the “Delegate to Agent” button to offload the actual implementation to the remote coding agent.

Not Happy?
Copilot creates a PR with code changes based on the issue. If you are unhappy with the results, you can supervise it further by leaving a comment “@copilot xxx”, which Copilot will pick up and obey. However, you should also consider why Copilot failed to do the work in a single shot. Did you give sufficient instructions in the issue? Better prompting could help. Did Copilot fail to understand how the system works? In that case, the context was not sufficient, as Copilot can’t infer everything. You should describe your system or subsystem better in a documentation file, and reference that in the issue or in a custom instruction (see also Space article A Happy Human is a Happy AI - Do You Give Sufficient Instructions to Your Developers and Your AI?).
Your mental image should be “the AI can write any code, I just need to support it in doing so.” When the AI fails, don’t say “the AI tool is bad,” but rather, “how can I improve the tool’s understanding to generate the correct solution?”

Agent Sessions, Your Future IDE?
In VS Code, there is a new orchestration view that combines your local agent sessions, your cloud agent sessions, and local command line sessions. While this is very rudimentary for now, I hope this evolves significantly next. This might become your main workspace view in the future. I can easily see this evolving to include better status on unittest coverage, E2E tests passed, security review executed, etc. Time will tell!



A Note on GitHub Copilot and World Dominance
There are many tools available. The concepts are largely the same across all of them. While Copilot definitely was not the best tool half a year ago, Microsoft’s infinite money and resources have helped here. Copilot has implemented most features from other tools, and in my opinion, you get the broadest capabilities in a single-stop fashion by using GitHub and GitHub Copilot: version control and actions, agentic IDE development, CLI mode, and remote coding agents. I only dislike their code review functionality and consider CodeRabbit or Codex clearly better at the moment.

That said, I don’t think Copilot is necessarily the best tool, and I’m sure many developers swear by Cursor, Claude Code, and similar tools. Great, use them! I am saying Copilot is the simplest to onboard to because it comes with batteries included. While the landscape can change quickly with new features and ideas popping up here and there, Copilot has shown it can embrace them relatively quickly. There is little need to jump on the “tool of the month” bandwagon.
