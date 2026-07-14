Everyone is optimizing their prompts. Boris Cherny threw the whole idea out.

He built Claude Code. He runs the team. And he uses the tool in a way that looks nothing like how most people use it. No secret model. No exotic config - he says it works great out of the box and barely customizes it. The leverage is not in the tool. It is in how he wraps his entire day around it.
Here is the part that breaks people's brains: his personal setup is almost boring. Five terminal tabs. A text file. A few slash commands. One rule he repeats more than any other. Put together, they let one engineer behave like a full team.
This is the actual method, pulled from his own January workflow thread, the Lenny and Pragmatic Engineer interviews, and Fortune's reporting. No theory. Just what he does.
PART 1 - HE RUNS AN ORCHESTRA, NOT A CHAT 

Five local, ten on the web, and one kicked off from his phone in bed.
Most people run one Claude and wait for it. Boris runs a swarm.
Five Claude Code sessions live in his MacBook terminal, tabs numbered 1 through 5, with system notifications telling him which one needs input. On top of that, 5 to 10 more run on the web at claude.ai/code. 
He hands sessions off from local to web, teleports them back with --teleport, and starts fresh ones from the Claude iOS app first thing in the morning - before he is even at a desk.
One detail most people get wrong: each local session gets its own git checkout, not a branch, not a worktree. That is how five agents edit the same repo at once without stepping on each other.
QUOTE:

"I don't prompt Claude anymore. I have loops that are running. They're the ones that are prompting Claude and figuring out what to do. My job is to write loops."
- Boris Cherny
Image
caption: 5 local + 5-10 web + phone. Separate git checkouts. One human conducting.

He is honest about the mess: 10 to 20 percent of those sessions get abandoned when they hit something unexpected. That is fine. When you run twenty, throwing away three costs nothing.
And he does not model-shop. Opus with thinking, for everything - even though it is bigger and slower per token. You steer it less, it is better at tool use, and across a whole task it wins. When you are orchestrating ten sessions, per-session speed stops mattering. You are never the one waiting. 
CALLOUT:

The bottleneck is no longer typing code. It is context switching between the agents writing it.
PART 2 - COMPOUND ENGINEERING: THE TEXT FILE THAT LEARNS

Every mistake becomes a rule. The rule is shared. The team gets smarter every week.
Boris's memory system is a plain text file called CLAUDE.md, checked into git. Not a vector database. Not a fine-tune. A file every Claude reads at the start of every session.
The twist is that it is a team object. His whole team writes to one shared CLAUDE.md multiple times a week. Anytime Claude does something wrong, the correction goes into the file so it never happens again. It holds style conventions, design guidelines, the PR template, the landmines.
QUOTE:

"Anytime Claude does something incorrectly, we add it to the CLAUDE.md so it knows not to do it next time."
- Boris Cherny
Image
Human spots the issue once. Claude writes the rule. Every future session avoids it.

His own personal CLAUDE.md? Basically two lines, pointing at the team's shared one. The team file sits around 2,500 tokens - deliberately lean, because he knows exactly how expensive a bloated one gets (more on that later).
The sharpest move is in code review. When Boris reviews a teammate's PR and spots an anti-pattern, he does not just leave a comment. He tags Claude on the PR through the GitHub Action and has it write the lesson straight into CLAUDE.md as part of that same PR. The fix and the rule ship together. This is what people mean by compound engineering: the codebase itself gets more intelligent with every merge.
PART 3 - PLAN FIRST, THEN GET OUT OF THE WAY 

The one-shot is a lie. The plan is the work.
When people watch Claude "one-shot" a feature and fail, they blame the model. Boris blames the missing plan.
His default is Plan mode - Shift+Tab twice. He does not let Claude touch code yet. He goes back and forth on the plan until it matches what is in his head. Only then does he flip to auto-accept edits and let it run.
QUOTE:

"If my goal is to write a Pull Request, I will use Plan mode, and go back and forth with Claude until I like its plan. From there, I switch into auto-accept edits mode and Claude can usually 1-shot it. A good plan is really important!"
- Boris Cherny 
Image
Plan mode -> agree on the plan -> auto-accept -> one-shot. The plan is where the thinking lives.
The reason one-shots fail for most people is bandwidth. You described a complex feature in a few words, so Claude built a different feature than the one in your head. Planning is how you raise the bandwidth before a single line is written. The "magic one-shot" is just a good plan cashing out.
PART 4 - THE INNER LOOP, TURNED INTO CODE

Anything he does more than twice becomes a command, a subagent, or a hook.
This is where the "vanilla setup" quietly becomes a machine. Boris does not re-prompt for routine work. He encodes it.
Slash commands for every inner-loop task, checked into .claude/commands/ so the whole team and Claude itself can use them. His workhorse is /commit-push-pr, run dozens of times a day. It uses inline bash to precompute git status up front, so the model does not waste a round trip figuring out state.
Subagents to protect context. Two he leans on: a Code Simplifier that cleans up after Claude finishes, and a Verify App agent with detailed instructions for testing end to end. A subagent runs, returns just the result, and keeps the mess out of his main session.
A PostToolUse hook that auto-formats every edit (bun run format). Claude's code is already clean; the hook handles the last 10 percent so nothing trivial breaks CI later.
Permissions instead of danger. He almost never uses --dangerously-skip-permissions. Instead he allowlists safe commands with /permissions (bun run build, bun run test, and friends), checked into .claude/settings.json and shared with the team. Everyone agrees on the safe defaults once.
MCP for the rest of his tools. Claude posts to Slack, runs BigQuery for analytics, pulls errors from Sentry - all through MCP, all configured in git. His warning: MCPs bloat your context and open a prompt-injection door, so have Claude review an MCP before you install it.
Image
Slash commands, subagents, hooks, permissions, MCP. The routine work stops being prompted and starts being infrastructure.
CALLOUT:

Boris's rule of thumb: anytime you do a task, build the thing that will do it for you next time. Intent compounds. Prompting does not.
PART 5 - VERIFICATION IS THE WHOLE GAME

is single most important tip, and it is not about prompting at all.
If you take one thing from how Boris works, take this: give the model a way to check its own work, and it will grind until the work is actually good.
QUOTE:

"Claude tests every single change I land to claude.ai/code using the Claude Chrome extension. It opens a browser, tests the UI, and iterates until the code works and the UX feels good."
- Boris Cherny
Image
No gate = the agent agrees with itself. A real check = 2-3x the quality.

A real feedback loop - a test suite, a type check, a build, a browser, a simulator - improves the final result by a factor of 2 to 3. Without it you do not have a loop. You have an agent grading its own homework and passing itself every time.
For long-running work, he layers the verification: prompt a background agent to check the result when the main one finishes, use an agent-stop hook for a deterministic gate, or run an autonomous looping plugin for the fully hands-off case. The point is the same at every scale - the loop only advances when reality says it did.
The payoff shows up in how his team spends its time. Because verification runs inside the loop, humans are freed to do the two things machines still cannot: review and steer. By the time an engineer opens a PR, the code is already in good shape.
PART 6 - TOKENMAXXING, DONE WITH EYES OPEN 

Spend more tokens. Just not the wasted ones.
Boris's advice to companies sounds reckless until you hear the second half.
QUOTE:

"Don't try to cost cut at the beginning. Give engineers as many tokens as possible."
- Boris Cherny
Image
Max the tokens that buy capability. Cut the ones lost to context bloat and re-read history.
He calls it tokenmaxxing. The bottleneck was never the cost of tokens - it is how fast engineers learn to use them well. Cap usage and you throttle the learning. And the number you should compare Claude Code to is not a 20-dollar coding assistant. It is what an engineer would have cost to do the same work. That is the benchmark.
But "spend more" is not the same as "spend blind." In a podcast breakdown, Boris walked through how tokens get burned before the model even reads your prompt - a big chunk lost to a bloated CLAUDE.md, another chunk paid to re-read old history. That is exactly why his own team file is a lean 2,500 tokens and his personal one is two lines. He maxes the spend that buys learning and cuts the spend that buys nothing.
PART 7 - THE HONEST PART

What the method does not fix.
None of this deletes the engineer. It moves him.
10 to 20 percent of Boris's sessions get abandoned. His code review became the new bottleneck the moment code-writing stopped being one. And the faster the swarm ships code nobody hand-wrote, the wider the gap between what is in the repo and what any human actually understands. A smooth workflow charges compound interest on that gap. The day you have to debug a system no one has read costs more than the tokens ever did.
Two engineers can run this exact setup and get opposite outcomes. One uses it to move faster on work they understand deeply. The other uses it to stop understanding the work at all. The tool cannot tell the difference. You can.
Image
Same five tabs, same CLAUDE.md, same commands. One path is leverage. The other is surrender.
Boris stopped writing code by hand. He stopped prompting turn by turn. He did not stop planning, reviewing, or steering - the parts that are actually him.
That is the method. Not better prompts. A better position: out of the typing, into the judgment.
