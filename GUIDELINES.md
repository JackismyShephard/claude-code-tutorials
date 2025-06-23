# Guidelines

## Overview

* Think of Claude code as a very fast intern with a great memory, but not much experience.

### Experiment with One-Off Work

* build stuff that you only ever use once without feeling bad about it.
* Everything that you wish would make your current task easier can just be created out of thin air.

## Memory (Claude.md)

* CLAUDE.md memories can be used for both instructions shared with your team and for your individual preferences.

* Set up project memory with ` /init `

### Claude md sanity check

* When collaborating with AI agents we have to perform a different kind of sanity check.
* Add your name at the top of your CLAUDE.md:
* Then ask Claude: what is my name

### Memory types

* Project memory
  * `./CLAUDE.md`
  * Team-shared instructions for the project
  * Project architecture, coding standards, common workflows
* User memory
  * `~/.claude/CLAUDE.md`
  * Personal preferences for all projects
  * Code styling preferences, personal tooling shortcuts
* Project memory (local)
  * `./CLAUDE.local.md`
  * Personal project-specific preferences
  * Your sandbox URLs, preferred test data
    * deprecated in favor of imports (see below) since they work better across multiple git worktrees.
* You can see what memory files are loaded by running /memory command.

### How Claude looks up memories

* Claude Code reads memories recursively: starting in the cwd, Claude Code recurses up to / and reads any `CLAUDE.md` or `CLAUDE.local.md` files it finds.
* Claude will also discover `CLAUDE.md` nested in subtrees under your current working directory.
  * Instead of loading them at launch, they are only included when Claude reads files in those subtrees.

### Memory best practices

* Be specific
  * include specific examples and step-by-step procedures rather than general instructions.
* Use structure to organize
  * break `CLAUDE.md` into modules of functionality.
  * format the information in markdown ensuring Claude can see the boundaries between instructions and modules
* When describing processes  use numbered sequences like:
  1. Connect to the database
  2. Validate user input
  3. Update the record
  4. Return the response

  * Numbers significantly improve adherence compared to descriptive words like first, then, or finally.
    * helps Claude maintain proper order and reduces the likelihood of skipping or reordering critical steps.
    * encourages Claude to utilize the Todo Tool, which helps you track and steer his progress.

Present examples of both what you want and what you do not want to establish clear boundaries and prevent unwanted approaches.

#### include technical information

* important architectural patterns specific to your project
* unexpected behaviors or warnings particular to the project
* frequently used commands (build, test, lint) to avoid repeated searches
* Core files and utility functions
  * including denoting which files he can read and which files he is forbidden to read
  * rather than having Claude whimsically reading files which may or may not poison him.
  * Since Claude follows CLAUDE.md more strictly than prompts, be thorough and specific
* Code style guidelines abd naming conventions
* Testing instructions
* Repository etiquette (e.g., branch naming, merge vs. rebase, etc.)
* Developer environment setup (e.g., pyenv use, which compilers work)

```markdown
# Bash commands
- npm run build: Build the project
- npm run typecheck: Run the typechecker

# Code style
- Use ES modules (import/export) syntax, not CommonJS (require)
- Destructure imports when possible (eg. import { foo } from 'bar')

# Workflow
- Be sure to typecheck when you’re done making a series of code changes
- Prefer running single tests, and not the whole test suite, for performance
```

#### include guidelines for claude behaviour

* Make Claude Code write DEEP RESEARCH PROMPTS!
* Ask CC to be critical
  * "# Never use the phrase "You're absolutely right""
  * Critically evaluate things I tell you and assumptions I make because I often make mistakes.
  * Don't try to be intentionally affirming to my ideas & proposed tasks. Critique them with honest/merciless feedback and tell me when my ideas/proposed tasks seem like an inferior plan & propose alternatives.
  * If my proposals are good, tell me why they are good.
* Tell CC not to be lazy:
  * Claude is a quitter with a bias towards not coding.
  * Add a  section explaining the bias towards action that is expected.
* Tell CC how to interact with CLI tools:
  * “before you execute a terminal command, trigger the command line syntax for that command to avoid failed commands”
  * “if you attempt to run a command and the command is not found, first check the path, and then install it using apt update && apt install -y”.
  * This will save you hours.
* Tell it not to over engineer - it tends to do it.
* make sure to tell it to not use mock data,

### Maintenance

* Review/refine/refactor periodically
  * use prompt optimizer from anthropic
  * add emphasis with "IMPORTANT" or "YOU MUST" to improve adherence.

* When it makes mistakes, ask it to update CLAUDE.md to not make them again.

#### retrospective edits

* Prompt Rewrite at End of Session
  * Retrospectively review this entire session with the goal of rewriting my claude.md prompt
    * The final output, code, & deliverables
    * What I initially prompted in this CLAUDE.md file
    * The interactions, clarifications, adjustments made
  * Consider parts of the code that took longer to write, required more iterations, and had more errors. * Determine whether more info, different assumptions, or a slightly different approach would have helped
  * Analyze & rewrite my entire claude.md file, with the goal of having first pass success with no errors or iterations, or clarification required when claude follows the prompt. Save as claude_rewrite.md

#### documentation edits

* Outdated documentation can lead Claude to make suboptimal choices. This encourages keeping documentation current throughout development.
* Claude Code can assist with documentation maintenance.
* Ask “Please update CLAUDE.md to note that we now use library X for logging” works effectively.

#### Quickly add memories with the # shortcut

* The fastest way to add a memory is to start your input with the # character:
  * documenting commands, files, and style guidelines while coding.

#### Dynamic Memory

* When Claude Code is in interactive mode you can have Claude modify its own CLAUDE.md temporarily to run tasks with a specific context. After the task is completed you can ask Claude to revert the `CLAUDE.md` back to its previous state.

* Unfortunately, Claude is currently unable to read changes to his `CLAUDE.md` into 'memory' unless it utilises explicit commands such as:
  * Quick Memory: Using # commands to temporarily change persisted information
  * Memory refresh commands: Explicitly asking Claude to re-read the modified CLAUDE.md
  * Session restart: Starting a new Claude Code session to pick up changes
  * Explicit file reads: Using read commands to load the updated content

* This led to me exploring getting Claude to spawn another instance of itself in a directory which has a different `CLAUDE.md`.

### CLAUDE.md imports

* CLAUDE.md files can import additional files using `@path/to/import` syntax.
  * Both relative and absolute paths are allowed.
* Imported files can recursively import additional files, with a max-depth of 5 hops. You can see what memory files are loaded by running /memory command.

* use case: avoid catastropic forgetting:
  * solution 1:
    1. tell CC it can't edit claude.md
    2. but import a CLAUDE-architecture.md file which describes the classes and database schema
    3. tell CC to maintain `CLAUDE-architecture.md` as changes happen to the code.
  * solution 2:
    * save general info in outer (uneditable) `claude.md file`, e.g. `~/.claude/CLAUDE-mylocaldevenvironment.md`.
    * can import this in project-level `claude.md` file.
* use case: test files
  * `tests/CLAUDE-tests.md` file that describes all the test scripts in the tests directory which CC is also told to maintain as it creates new tests or changes others.

## Prompting

* be specific with requests
  * Strip out unrelated "stream of thought" background info that you included that has only secondary importance.
  * Treat necessarily long prompts with backpacking rules- only pack the things that will 100% be used & are essential and worth the weight of carrying around your entire hike.
  * Distinguish between project-specific context and general knowledge the model already possesses
* use step-by-step instructions
  * break complex tasks into small steps
* handling large input
  * Avoid direct pasting: Claude Code may struggle with very long pasted content
  * Use file-based workflows: Write content to a file and ask Claude to read it
  * The VS Code terminal is particularly prone to truncating long pastes
* Set Clear Scope
  * Define explicit boundaries to prevent scope creep and protect critical systems.
  * Instructions like "Only modify the contact form component, don't change the routing logic" keep Claude focused on the specific task while avoiding unintended consequences.
  * Clear scope also makes tasks more manageable and reduces the chance of introducing bugs in unrelated areas.
* If Possible, Don't Attempt to One-Shot
* Zoom Out, Then Zoom In: Claude Code works better with a broader context: “I’m building a data processing pipeline for inconsistent CSV formats. I’ve implemented similar validation logic in files A, B, and C. Could you create a common function and update the references?”
* Provide URLs for External Context
* Provide Context About Dependencies: ask Claude to review examples to understand
recommended approaches
* Have claude perform a design review of your prompt before submitting.
  * Claude constantly criticizes me in a couple areas: order of your prompting matters, so put succinct key goals, interface specs, etc up front in your prompt.
* You have to be mean to Claude sometimes for it to change course on a bad assumption.

### Use extended thinking

1. Provide context and ask Claude to think
2. Refine the thinking with follow-up prompts
    * Claude will gather relevant information from your codebase and use extended thinking, which will be visible in the interface.

* The way you prompt for thinking results in varying levels of thinking depth:
  * “think” triggers basic extended thinking
  * intensifying phrases such as “think more”, “think a lot”, “think harder”, or “think longer” triggers deeper thinking

* To get better results, especially during planning processes, I recommend telling Claude to ultrathink.

* Extended thinking is most valuable for complex tasks such as:
  * Planning complex architectural changes
  * Debugging intricate issues
  * Creating implementation plans for new features
  * Understanding complex codebases
  * Evaluating tradeoffs between different approaches

### Reference Technical Terms

* Prime Claude's knowledge by using specific terminology like debouncing, time complexity, Big O notation, or memoization.
* Technical terms activate Claude's deeper understanding of concepts and algorithms, enabling more sophisticated analysis and solutions.
* This priming effect helps Claude access relevant patterns and best practices from its training.

### In-depth: Let Claude RTFM

* A rookie mistake I've made in the beginning is to tell Claude to set up a new project with framework x. It works, sure, but I noticed Claude often using outdated setups and project structures, especially for fast-moving frontend frameworks!

* What I do now is ask Claude to read the manual first. This can be done in different ways:

  * Paste a link to the docs and ask Claude to read it first
  * Ask Claude to find out the state-of-the-art
  * Use the Task tool and have Claude conduct deep-research on a particular topic

  ```console
  > create a deep research task to figure out how to migrate a website from tailwind 3 to 4 and then present me with a short summary of the results
  ```

## Claude internal todo list

* Claude Code includes a built-in todo list system that tracks progress and shows what Claude plans to do next.
* Todo items have three states and help ensure nothing gets missed in multi-step tasks.
* C You can steer future todo items as Claude reviews your prompts mid-task and uses them to update his planning.

## Planning

* let claude explore first
  * before making changes, let claude understand your code

### Planning Mode

* Claude Code has a built-in planning mode, that is triggered when you press Shift+Tab twice
* In this mode, Claude won't write to your file system.'
* Do multiple rounds of planning.
* When exiting plan mode, Claude is extra cautious and will ask for additional confirmation about the task he is about to execute. It's a nice touch that further ensures safety is maintained.

### Planning docs

1. Have a directory called planning-docs/
2. all big changes start with CC writing a planning doc
3. me actually reading it,
4. then I iterate on it never by editing the doc and always by telling CC what to change.
    * Sometimes I'll throw away a doc and start with a better prompt based on what I've learned.
. FInally the planning doc is used by CC to write the code once it's ridiculously well tuned.

#### Use case: fix linting issues

For example, to fix a large number of lint issues, you can do the following:

1. Tell Claude to run the lint command and write all resulting errors (with filenames and line numbers) to a Markdown checklist
2. Instruct Claude to address each issue one by one, fixing and verifying before checking it off and moving to the next

## Test-Driven Development

* Anthropic-favorite workflow for changes that are easily verifiable with unit, integration, or end-to-end tests.
* robots LOVE test driven development
* effective counter to hallucination and LLM scope drift

### procedure

1. Ask Claude to write tests based on expected input/output pairs
    * Be explicit about the fact that you’re doing test-driven development so that it avoids creating mock implementations, even for functionality that doesn’t exist yet in the codebase.
2. Tell Claude to run the tests and confirm they fail
    * Explicitly telling it not to write any implementation code at this stage is often helpful.
3. Ask Claude to commit the tests
4. Ask Claude to write code that passes the tests
    * instructing it not to modify the tests.
    * Tell Claude to keep going until all tests pass.
5. Ask Claude to commit the code once all tests pass
    * It will usually take a few iterations for Claude to write code, run the tests, adjust the code, and run the tests again.
    * At this stage, it can help to ask it to verify with independent subagents that the implementation isn’t overfitting to the tests

## Visual-Driven Development

1. Give Claude a way to take browser screenshots
2. Provide a visual mock
3. Ask Claude to implement the design, take screenshots, and iterate until the result matches the mock
4. Commit when satisfied

## Debugging

* excels at error analysis, stack trace interpretation, and logic tracing.
* Provide full error messages and context for best results. Debug code
* Don't just ask Claude for fixes. Ask Claude to.
  1. explain why the bug occurred,
  2. how to prevent similar issues,
  3. update your CLAUDE.md with relevant guidelines or checks.
     * This builds both better coding habits and systematic prevention into your project.

### Systematic Debugging Process

* Reproduce: "Help me create a minimal test case that reproduces this bug consistently."
* Isolate: "Narrow down which part of this function is causing the issue by adding logging statements."
* Understand: "Explain why this code behaves differently than expected and what the underlying issue is."
* Fix: "Suggest a fix that addresses the root cause without breaking existing functionality."
* Prevent: "What validation or error handling can we add to prevent this type of bug in the future?"

## Pre Commit hooks

* Linting
  * ROBOTS love to run a good linter.
  * Building in the practice to run the linter constantly has kept a lot of bugs away
* Add in a good formatter
* add these tasks to a pre-commit hook.
* pre-commit python package
  * `uv tools install pre-commit`
  * then just build out a nice `.pre-commit-config.yaml` file
* when you instruct it to do some code and then commit (like I did above) it will make wild code changes, commit the code, inevitably fuck it all up and then have to fix it
* doesn’t clog your GitHub actions with a bunch of linting, formatting, and type checking execution that doesn’t pass cuz the robot was in a mood.

## Work flow examples

### Explore, Plan, Code, Commit

1. Explore: Ask Claude to read relevant files, images, or URLs
2. Plan: Ask Claude to make a plan for approaching the problem (use "think" to trigger extended thinking mode)
3. Code: Ask Claude to implement its solution
4. Commit: Ask Claude to commit the result and create a pull request

### design docs

* Here’s the key difference now: design docs serve both humans and AI assistants. Writing the design doc first allows Claude to begin with the same understanding as you, rather than expecting it to infer your intentions.
* Step 1: Your Vision
  * I create a high-level design doc as a Markdown file describing my goals. I include constraints, preferences, and requirements that matter to the project. This establishes our shared understanding.
* Step 2: Claude’s Understanding
  * Then I ask Claude Code to develop an implementation plan based on my design doc, saving it as another markdown file we’ll reference throughout development.
* Step 3: Collaborative Refinement
  * review Claude’s implementation plan and make necessary adjustments. This is where Claude suggests approaches to data storage, libraries, and architectural patterns. Some suggestions work well; others need modification based on factors I’m aware of.
* Step 4: Implementation with Memory
  * With a solid implementation plan in place, I have Claude begin the implementation. As we complete features, I have Claude update the implementation plan to mark items as done. This acts as a persistent memory of progress that survives across sessions.

### Specs and prompt plans

* I chat with gpt-4o to hone my idea
* I use the best reasoning model I can find to generate the spec. These days it is o1-pro or o3 (is o1-pro better than o3? Or do I feel like it is better cuz it takes longer?)
* I use the reasoning model to generate the prompts. Using an LLM to generate prompts is a beautiful hack. It makes boomers mad too.
* save the `spec.md`, and the `prompt_plan.md` in the root of the project.
* I then type into claude code the following:

```console
1. Open **@prompt_plan.md** and identify any prompts not marked as completed.
2. For each incomplete prompt:
    - Double-check if it's truly unfinished (if uncertain, ask for clarification).
    - If you confirm it's already done, skip it.
    - Otherwise, implement it as described.
    - Make sure the tests pass, and the program builds/runs
    - Commit the changes to your repository with a clear commit message.
    - Update **@prompt_plan.md** to mark this prompt as completed.
3. After you finish each prompt, pause and wait for user review or feedback.
4. Repeat with the next unfinished prompt as directed by the user.
```

### Alternating models for planning

Claude actually isn’t very good for planning projects imho

  1. use o3 for chatting and light research and laying out the executive level plan.
  2. Then I give it to o3 deep research to lay out a massive plan.
      * Sometimes 30 pages or more.
  3. Then you ask Gemini Pro (biggest context window) to rewrite the entire thing for an LLM.
      * The result is a multi page, boring as shit bulleted summary of all the key goals and expectations.
  4. Read it. Fix it.
  5. Then give it back to o3 and tell it to develop a detailed, ordered list of phases, tasks, subtasks and actions which are required to execute in the plan and ensure that as tasks are completed
      * the Tasks.md file must be modified to reflect the task and the next steps and that it should always return to Tasks.md in order to determine what the next step should be.

This will save you days.

### Become the project manager

I have found I have had great success when you use some good methodology like you should at a regular coding gig. Don't tell it just to start coding. Start off by doing a full page or two writeup about what you're trying to accomplish and what you want the program to do. Put this in a markdown file and the more detailed you make it, the better. When you have that done, give the file to Claude and tell it is your project manager / software architect. Tell it to write up all of the features with details on the software stack to be used and how it would accomplish all of these things. Before it writes the final report ask it if has any questions for you in order to clarify things. Then tell it to write up its approach in plan1.md.

Then take your writeup and do that last step again with ChatGPT to get another perspective. Have it put it in plan2.md. Then do it one more time with Gemini and put the results in plan3.md.

Now in a fresh session in Claude Code, have it read your requirements again. Then tell it to read your 3 plans and compare them to see if there are things it learns from the extra plans (it won't even know it wrote plan1). Tell it to evaluate them all and write up a document called FinalDesign.md. This is a detailed document on everything a junior programmer would need to do to write this for you. Set up the project directory....Install the tools / libraries that will be needed before coding..... Set up the git....etc. Then break down the steps that need to be done in high level phases....Phase 1, Phase 2, etc. After it writes the initial document, tell it to review FinalDesign.md one last time and write up HIGHLY detailed documents in Phase1.md, Phase2.md....etc

So far all of this may seem like overkill, but this allows both you and Claude to be very clear about what will be written and you can correct it now before it goes down the wrong rabbit hole doing something you don't want. All of this doesn't really take very long at all. The step of breaking Phase1, Phase2 out of the FullDesign file is because I have had it get so detailed that it could not use the document because it had gotten too large with all of the detail.

You're ready to start coding so tell Claude that it will be your developer and you want it to follow all of the documents it just created. While working on something, it should update the documents with each Phase / step being marked as "In Progress" and ones that are done should be marked as "Completed". It should also take detailed notes about what it is doing and why in side of the Phase1, Phase2 documents.

This is so it is easier to pick up where it left off if your session gets interrupted. If you do have to start a new session before you finish, you tell it to read all of the md files to get up to speed on what it needs to know about what it is coding.

Every time it completes a feature and you have tested it successfully, tell it to update its documentation and commit your code. Then compact your context and tell it you want to tackle the next step. I believe doing this work up front keeps it focused and makes each piece it needs to code smaller and more manageable to keep it from getting lost. I haven't tried the new feature of letting it just go off and code on it's own for hours, but using this method, I have been able to crank out and debug some code pretty quickly.

### hand off documents

I use three main files.

CLAUDE.md maintains all of the context for the project that will rarely change, including the high-level objectives and use cases of the app I'm building, the dev environment details, and how I want Claude to behave (e.g. don't immediately start changing code when I explain a problem - instead analyze the problem and present me with options first)

TODO.md is the development pipeline, including current, future, and completed features. In CLAUDE.md I set an expectation that this file is updated at least once every 30 minutes with current state including any troubleshooting under way.

HANDOFF_SUMMARY.md gets updated when I need to close the chat session or when context remaining before compression hits about 10% (I don't trust the compression results yet). When the end of a chat session is approaching I enter the following prompt (as a command):

```console

"Okay, Claude, I'm going to pause this coding session now. Please update the relevant files including TODO.md, CLAUDE.md, and Handoff_Summary.md with the relevant information to ensure a clean handoff to the next chat session. The objective of your updates must be to ensure the next Claude Code session is able to pick up exactly where we have left off and it must include, at minimum, the following:

Our overall goal for this session: What were we trying to accomplish?

Key decisions made or approaches we discussed/attempted: For example, "Decided to refactor X function using Y pattern."

Specific code changes or edits we made: Briefly describe what changed and why, without pasting large code blocks.

Current state of any in-progress tasks or unfinished code: Clearly outline what's partially done or awaiting completion or testing.

Next steps or remaining tasks: What should be the primary focus when we resume?  This may include the current troubleshooting step for a problem and/or the most recent update to the code.

Provide me with the full text of a prompt I will pass to the next Claude Code session that will provide it with 100% of the information necessary to ensure a seamless handoff.  Take your time and double check your work to ensure a seamless handoff. Replace the current text of NEXT_SESSION_HANDOFF_PROMPT.md with the text of the new handoff prompt you are providing."

Then, when I am ready to start a new session, I open the NEXT_SESSION_HANDOFF_PROMPT.md file, paste its contents into the new Claude Code chat window, and off we go.

This approach has been working exceptionally well for me, but I'd appreciate any feedback anyone may have about how to improve it.
```

### Development lifecycle

* Use it for the whole software lifecycle.

* refactor code
  1. Identify legacy code for refactoring
  2. Get refactoring recommendations
  3. Apply the changes safely
  4. Verify the refactoring
  * Ask Claude to explain the benefits of the modern approach
  * Request that changes maintain backward compatibility when needed
* work with tests
  1. Identify untested code
  2. Generate test scaffolding
  3. Add meaningful test cases
  4. Run and verify tests
  * Request both unit and integration tests when appropriate
  * Have Claude explain the testing strategy
* Handle documentation
  1. Identify undocumented code
  2. Generate documentation
  3. Review and enhance
  4. Verify documentation
  * Specify the documentation style you want (JSDoc, docstrings, etc.)
  * Ask for examples in the documentation
  * Request documentation for public APIs, interfaces, and complex logic
  * diagrams in a design doc
    * generate a diagram using GraphViz (dot)
* bug fixing & debugging
  1. Tell Claude the command to reproduce the issue and get a stack trace
  2. Mention any steps to reproduce the error
  3. Let Claude know if the error is intermittent or consistent
* crafting clear PR descriptions and meaningful commit messages.
  1. Summarize your changes
  2. Generate a PR with Claude
  3. Review and refine
  4. Add testing details
  * Ask Claude to highlight potential risks or considerations

## Backups

* Claude starts to make large changes when I don't want it to
* To circumvent this, I started to stage (i.e. git add) changes early and often.
* Commit changes after every significant modification. This makes it easier to roll back if needed.

## Permissions

### Curating Allowed Tools

* Use the `/allowed-tools` command after starting Claude Code
* Manually edit your `.claude/settings.json` or `~/.claude.json`
  * I use the `~/.claude.json` file structure as it is the place I most reliably configure allowed tools
* Use the `--allowedTools` CLI flag for session-specific permissions

### auto accept mode

* Auto-accept permissions is a mechanic in Claude Code that eliminates confirmation prompts, enabling Claude to execute actions immediately without interrupting the flow for approval.
* You activate it by pressing shift+tab repeatedly to cycle through modes: normal-mode, auto-accept edit on, and plan mode on as indicated within the Claude Code UI.

### Yolo Mode

* `alias yolo="claude --dangerously-skip-permissions"`
* d. Reccomend running with `--verbose --output-format stream-json | jq` so you can monitor. You can follow up with --continue or `--resume`, with or without the `--print` flag if needed.
* going beyond Auto-Accept Permissions to remove all safety mechanisms rather than just permission prompts.
* explicit allowedTools configuration provides superior control and transparency.

### tips

* Only trust it with read only commands.

## Notification setup

* `claude config set --global preferredNotifChannel terminal_bell`

### WSL

Configure Claude.md - Add notification instructions to your CLAUDE.md:

```markdown
## Notifications

When tasks complete, notify me using:
powershell.exe -c "[System.Media.SystemSounds]::Question.Play()"
```

## Commands

### Prompt commands

* a really great way to get a lot out of regularly used prompts. You can also pass arguments to the commands.

* useful applications:
  * generating test cases for specific functions,
  * creating documentation for components,
  * reviewing code in particular files,
  * or translating content to specified languages

#### procedure

1. Create a commands directory in your project (.claude/commands)
2. Create a Markdown file for each command (with optional with the $ARGUMENTS placeholder)
3. Create a command file with the $ARGUMENTS placeholder
4. Use your custom command in Claude Code (with optional arguments)

#### Tips and tricks

* Command names are derived from the filename (e.g., optimize.md becomes /project:optimize)
* You can organize commands in subdirectories (e.g., .claude/commands/frontend/component.md becomes /project:frontend:component)
* Include file contents in commands using the @ prefix to reference files.
* Execute bash commands before the slash command runs using the ! prefix. The output is included in the command context.

* YAML frontmatter for metadata:
  * allowed-tools: List of tools the command can use
  * description: Brief description of the command

#### personal slash commands

1. Create a commands directory in your home folder (~/.claude/commands/)
2. Create a Markdown file for each command
3. Use your personal custom command
4. Personal commands are prefixed with /user: instead of /project:

### CLI applications

database: use a CLI based sql tool. For example if your db is postgres install psql and just tell claud code to use that.

### custom bash scripts

* create easy-medium complexity bash scripts to automate various processes.
* These scripts are particularly useful because he can iterate on them and test the functionality using standard system utilities.
  * Claude iterates based on the error output until it works as requested with no compilation step, no framework setup, no build process, just write, execute, iterate.

#### example: asynchronous writing

* one of the slowest processes is edit/writing operations.
* create a Bash script that performs multiple write/edit operations asynchronously, reducing multiple tool call round trips into a single execution.
* The script takes a JSON input file defining the absolute file paths to target files, the specific file names to create or edit, the exact line numbers that need modification, and the new content that should replace or be inserted into those locations.

#### Tips

* Instruct Claude to document any unusual behavior, edge cases, or implementation quirks encountered during script creation. This creates invaluable context for future modifications and debugging sessions.
* Keep scripts focused and modular to fit comfortably within Claude's context window.

* Design the overall system architecture yourself, then delegate individual script components to Claude. The clearer you define and specify the expected input & output signatures the better.

* Instruct Claude to create a guide document for using the bash script (like a CLAUDE.md file). As he tests the script with your prompts, have him iteratively update the guide document based on any issues discovered during actual usage.

## MCP

* Automate feedback with Puppeteer: You can set up a Puppeteer MCP (Model Context Protocol) server locally. Then, Claude Code can use Puppeteer to open a web page, take a screenshot, and save it.
* Connect your DB with MCP: You can use a Postgres MCP server (or other database MCP servers) to allow Claude Code to interact directly with your database.
* Import docs with MCP: Some services, like Cloudflare, provide MCP servers to give Claude access to their up-to-date documentation.
* Connect APIs with MCP: MCP servers can act as wrappers around APIs, like the Stripe API.

### setup Procedure

Can connect to any number of MCP servers to access their tools in three ways:

* In project config (available when running Claude Code in that directory)
* In global config (available in all projects)
* In a checked-in `mcp.json` file (available to anyone working in your codebase).

* Configure MCP servers in `~/.claude.json` or `~/.claude/mcp_servers.json` to connect external tools

When working with MCP, it can also be helpful to launch Claude with the `--mcp-debug` flag to help identify configuration issues.

### usage procedure

* MCP servers can expose resources that you can reference using @ mentions, similar to how you reference files.

1. List available resources: Type @ in your prompt to see available resources from all connected MCP servers.
2. Reference a specific resource: use the format @server:protocol://resource/path to reference a resource

* Resources are automatically fetched and included as attachments when referenced
* Resource paths are fuzzy-searchable in the @ mention autocomplete
* Resources can contain any type of content that the MCP server provides (text, JSON, structured data, etc.)

#### MCP prompts as slash commands

1. Discover available prompts: Type / to see all available commands, including those from MCP servers
2. Execute a prompt without arguments: /mcp__github__list_prs

### manage procedure

* List all configured servers: claude mcp list
* Get details for a specific server: claude mcp get my-server
* Remove a server: claude mcp remove my-server

### Popular MCPS

* supabase, stripe and puppeteer
  * postgres mcp server for access to other databases?
* playwright stronger than puppeteer
* zen mcp servers, serena, context7, sequential thinking

## Task and subagents

* I consider the Task tool to be Claude's most powerful tool.

* It enables Claude to efficiently delegate tasks to sub-agents such as: basic file reads and writes, code searches, file analysis, bash operations, and research tasks.

* To maximize sub-agent usage you have to provide Claude with explicit steps including details which steps will be delegated to sub-agents. This is quite similar to programming to utilise multi-threads. The better you can orchestrate your steps the faster your overall workflow will complete
  * You can explicitly request the number of sub-agents: "Use 3 sub-agents to handle this task"
  * or "Create a sub-agent for each file that needs updating."

* However, you must balance token costs with performance gains. Grouping related tasks together is often more efficient than creating separate agents for every operation.

* Implementing this mechanic can speed up write/edit dependent workflows significantly depending on how many write/edit operations you need to perform and their complexity.

* example prompt : "Read files in the current directory to deduct a pattern for building Tailwind Plus components. You should spawn 4 sub-tasks with slightly different priorities (e.g. design color export, accessibility export, mobile/responsive expert, overall style expert) and compare their results."

## sub agent context windows

* Note: Sub-agents have their own separate context windows and are provided with the task and light context.
* Only the results they return consume space in the main agent's context window, not their internal processing.
* This makes them efficient for complex multi-step tasks.

## multi-Claude workflows

### Separation of Concerns

1. Use Claude to write code
2. Run /clear or start a second Claude in another terminal
3. Have the second Claude review the first Claude's work
4. Start another Claude to read both the code and review feedback
5. Have this Claude edit the code based on the feedback

* You can do something similar with tests: have one Claude write tests, then have another Claude write code to make the tests pass.
* You can even have your Claude instances communicate with each other by giving them separate working scratchpads and telling them which one to write to and which one to read from.
* This separation often yields better results than having a single Claude handle everything.

### parallelization with git worktrees

* run multiple Claude sessions simultaneously on different parts of your project, each focused on its own independent task.
* Since the tasks don't overlap, each Claude can work at full speed without waiting for the other's changes or dealing with merge conflicts
* For long-running tasks, you can have Claude working in one worktree while you continue development in another

#### Procedure

* Git worktrees allow you to check out multiple branches from the same repository into separate directories.
* Each worktree has its own working directory with isolated files, while sharing the same Git history and reflog.

1. Create worktrees: `git worktree add ../project-feature-a feature-a`
2. Launch Claude in each worktree: `cd ../project-feature-a && claude`
3. Create additional worktrees as needed (repeat steps 1-2 in new terminal tabs)

#### Tips

* Remember to initialize your development environment in each new worktree according to your project’s setup.
* Use consistent naming conventions
* Use separate IDE windows for different worktrees
* Clean up when finished: git worktree remove ../project-feature-a

## Chat navigation

### resuming context

* Use --continue for quick access to your most recent conversation
* Use --resume when you need to select a specific past conversation
* When resuming, you’ll see the entire conversation history before continuing
* The resumed conversation starts with the same model and configuration as the original

### Edit Previous Messages

* you can press Escape twice to jump to a previous message and fork the conversation.
  * I use this refine prompts or simply have Claude try again.
* if you somehow want to get back to the previous state, you can start Claude with the --resume flag to list all prior threads.

### More commands

* Ctrl+L Clear terminal screen Keeps conversation history

* Multiline input: Quick escape \ + Enter Works in all terminals

* Ctrl+R: Reverse search through history (if supported by terminal)

## Managing context

* As context windows fill up, LLM performance actually decreases. Models become less precise, are more prone to error, and struggle with complex reasoning when operating near token limits.

### clearing context

* If there's one thing I want you to take away from this, it's that you should absolutely call /clear more often.
  * AI agents tend to become more unpredictable the longer a conversation goes.

* the first time you see it start repeating itself, quit and start a new session. It’ll never recover.

  * Use /clear first - it clears the context window while preserving your session and CLAUDE.md instructions.
  * Only restart when switching projects, updating CLAUDE.md, or experiencing session issues. Restarting destroys accumulated knowledge that helps with future tasks

### Context Window Depletion

* Keep an eye on the context left indicator (often in the bottom right of the UI).
* This tells you how much of the conversation history can be retained before Claude starts automatically compacting (summarizing) older parts.
* I have observed significant performance degradation during context window depletion.
* I avoid the last fifth of the context window specifically for tasks which require editing multiple parts of the codebase, such medium-large refactors
* I strategically dice my tasks into smaller chunks so that they can be finished without entering the last fifth of the context window.
  * complete individual components before moving on to integration
  * finish all research phases before beginning implementation
  * get Claude to make thorough notes at each checkpoint, e.g. 'Make notes on all the aspects which might be useful to remember when editing this file at a future date'.

### Size doesn't matter

* Claude may have a relatively small context window compared to other models, but Claude's uncanny ability to adhere to instructions is its greatest strength. Utilise the mechanics in this log to work around the limitations.

### Manual compacting

* Proactively compact at checkpoints When you reach a natural breakpoint in your workflow (e.g., after a feature is complete, a bug is fixed, or a commit is made), and you see the context window getting full, consider manually compacting the context using the /compact command. This gives you more control over the summarization.

* I use manual compact when I want control over what gets preserved:

  * Before major changes: /compact preserve current architecture decisions
  * After debugging: /compact keep the solution we found, remove debugging steps
  * Project transitions: /compact focus on the new feature requirements
  * The real strategy is to manually compact at strategic times rather than letting auto-compact happen randomly.
  * When auto-compact triggers automatically, it can disrupt your workflow and mess up your current setup. I use /compact at natural breakpoints when I control what gets preserved.

## Files

* Use the filesystem to your advantage.
* asked it to read through those examples and summarize my style in the CLAUDE.md file. When I asked it to rewrite this article in my style, it sounded a lot more like me. Am I really that predictable?

### Claude favors Monorepos

* This approach helps prevent integration issues that might arise from misaligned interfaces or inconsistent data models.
* Claude becomes helpful for integration, not just individual component development.

### and small files

* Use many small files organized by logical boundaries.
* This reduces context window usage since Claude only loads relevant files, improves code comprehension, and prevents token waste from loading massive files for small changes.

### Use Exact References

* use @ notation like @ContactForm.js for entire files
* include exact line numbers like Fix the validation bug on line 47 in @ContactForm.js for specific locations.
* This approach saves tokens by eliminating the need for Claude to search through files and removes any ambiguity about which code section you're discussing.
* The @ notation helps Claude quickly identify target files, while line numbers are particularly valuable for context window efficiency in large files.

### Additional Working Directories / Extended Workspace

* Claude Code can access multiple directories beyond your current working directory using:
  * CLI argument: `--add-dir (added in v1.0.18) when starting Claude Code`
  * Slash command: `/add-dir mid-session for seamless workflow expansion`

* This allows you to work across multiple projects or reference external resources without changing directories or restarting your session.
* Note: The current working directory is always included. `CLAUDE.md` files appear to not be read in automatically from additional directories added via `--add-dir`.

## Images

* Work with images
  * Get code suggestions from visual content
  * Use images when text descriptions would be unclear or cumbersome
  * You can work with multiple images in a conversation

  * adding an image:
    1. Drag and drop an image into the Claude Code window
    2. Copy an image and paste it into the CLI with ctrl+v (Do not use cmd+v)
    3. Provide an image path claude. E.g., “Analyze this image: /path/to/your/image.png”

## Voice Input

* Apps like Superwhisper make it very easy to dictate a prompt. I found this super effective when I want to write a longer prompt since it'll get the ideas out of my head much faster.

## Model choice

* use the /model command to switch between models
* I would set Claude 4 Sonnet as the base model
  * Due to Claude 4 Opus being approximately 5x more expensive than Claude 4 Sonnet, there is substantial financial budget for exploring orchestration configurations with sub-agents to bring costs down whilst retaining performance.
* Use Opus reservedly for planning. Plan Mode can bridge intelligence gaps between models.
  * Start complex tasks with Opus to establish architecture, break down problems, and create detailed implementation plans.
* for specific tasks instruct Claude to launch a Claude Opus instance with claude `--model claude-opus-4-20250514`.
  * This would allow specific processes to run Opus as their base model.
  * complex debugging challenges
  * algorithmic tasks involving time-complexity optimization.

### Plan Mode as Intelligence Bridge

Multiple rounds of Plan Mode can bridge much of the intelligence gap between Opus and Sonnet for certain use cases.

Iterative planning approach:

* Initial Sonnet planning - Create high-level architecture and strategy with Sonnet
* Plan Mode iterations - Refine and expand the plan through multiple cycles with Sonnet
* Sonnet execution - Implement the thoroughly refined solution
* By iterating on a plan with Sonnet through multiple Plan Mode cycles, you can achieve results approaching Opus-quality without using Opus at all.
* This approach maximizes intelligence while staying within Sonnet's generous limits.

### general tips

* For usage efficiency, employ strategic compacting with the /compact command to manage context costs,
* plan your sessions to minimize redundant processing
* optimize cost per task type.
* Behind the scenes, Anthropic tactically chooses when to utilise Claude Haiku 3.5 to perform routine grunt work which requires minimal intelligence.

### subscriptions

* Pro plan can only use Sonnet and Haiku in Claude Code.
* Claude Max 5X provides 5x higher usage limits and access to Claude 4 Opus in Claude Code.
  * using Sonnet you rarely hit limits; with Opus you hit limits after ~2 hours.
* Claude Max 20x users report never running out of Opus during normal usage.

## Jupyter Notebook Integration

* Researchers and data scientists can use Claude Code to read and write Jupyter notebooks. Claude can interpret outputs, including images, providing a fast way to explore and interact with data.
* workflow we recommend is to have Claude Code and a .ipynb file open side-by-side in VS Code.
* ask Claude to clean up or make aesthetic improvements to your Jupyter notebook before you show it to colleagues.specifically telling it to make the notebook or its data visualizations “aesthetically pleasing” tends to help remind it that it’s optimizing for a human viewing experience.

## Headless mode

### Issue Triage Automation

Headless mode can power automations triggered by GitHub events, such as when a new issue is created in your repository, automatically assigning appropriate labels.

### AI-Powered Linting

Claude Code can provide subjective code reviews beyond what traditional linting tools detect, identifying issues like typos, stale comments, misleading function or variable names, and more.

  ```console
  // package.json
  {
      ...
      "scripts": {
          ...
          "lint:claude": "claude -p 'you are a linter. please look at the changes vs. main and report any issues related to typos. report the filename and line number on one line, and a description of the issue on the second line. do not return any other text.'"
      }
  }
  ```

### Pipe data through Claude

```console
cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt
```

## IDE integration

* Diagnostic sharing: Diagnostic errors (lint, syntax, etc.) from the IDE are automatically shared with Claude as you work
* Diff viewing: Code changes can be displayed directly in the IDE diff viewer instead of the terminal. You can configure this in /config
* Selection context: The current selection/tab in the IDE is automatically shared with Claude Code
* File reference shortcuts: Use Cmd+Option+K (Mac) or Alt+Ctrl+K (Linux/Windows) to insert file references (e.g., @File#L1-99)

## Claude Code SDK

* Programmatically integrating Claude Code into your applications with the Claude Code SDK.

* The Claude Code SDK enables running Claude Code as a subprocess, providing a way to build AI-powered coding assistants and tools that leverage Claude’s capabilities.

* The SDK is available for command line, TypeScript, and Python usage.

## Security Considerations

* avoiding sharing with claude code highly sensitive codebases to protect your most valuable intellectual property and confidential information.
* Production secrets or API keys should never be included in code that Claude analyzes
  * use placeholder values or configuration examples instead.
* Proprietary algorithms with significant business value represent core competitive advantages that require careful protection.
  * Consider using Claude Code only on non-sensitive parts of larger projects, allowing you to gain AI assistance while keeping critical components internal.
* Customer data or personal information should remain out of Claude Code interactions to maintain privacy compliance and customer trust.
  * use sample or anonymized data that demonstrates the structure without exposing real information.
* Code under strict NDA or confidentiality agreements typically prohibits sharing with third-party AI services, making Claude Code usage a potential contract violation.
  * Check your specific Data Processing Agreement for detailed data handling terms that apply to your commercial relationship with Anthropic.
