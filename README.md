# skills

These are the Swift and SwiftUI conventions I give LLM coding agents (Claude Code, Codex) when they write code for my apps. I plan each app's architecture before the first commit; these files hold the patterns that stay the same from app to app, so the generated code reads like code I wrote by hand. Each project's `CLAUDE.md` or `AGENTS.md` tells the agent to read this folder before it starts.

The examples come from my own apps: the state and presenter code is from [Avocadough](https://github.com/SparrowTek/avocadough), a Bitcoin Lightning wallet, and the networking and preview samples are from Effem, a podcast player.

## Built with these rules

[Perch BTC](https://apps.apple.com/us/app/perch-btc/id6800058660), a Stacks staking app, went from first commit to the App Store in under seven weeks, built with Claude Code against these files. It is SwiftUI and SwiftData organized as views, presenters, and state, with chain data from [StacksKit](https://github.com/SparrowTek/StacksKit), my open-source Swift port of the Stacks.js core.

## The rules

| File | What it covers |
|---|---|
| [Architecture.md](Architecture.md) | Views and presenters instead of view models. `@MainActor @Observable` state objects in the environment own routing, sheets, and full-screen covers through enums, and child states keep an `unowned` reference to their parent. |
| [Concurrency.md](Concurrency.md) | No bare `Task {}`. Async work starts from SwiftUI's `.task(id:)` through a `TaskTrigger`, so it is tied to the view's lifetime and is cancelled with it. |
| [Networking.md](Networking.md) | All HTTP goes through [NetworkingKit](https://github.com/SparrowTek/NetworkingKit): an `EndpointType` enum per API and one service per group of related endpoints. |
| [SwiftData.md](SwiftData.md) | A versioned schema behind a `typealias`, a `setupModel(inMemory:)` modifier for the container, and `@ModelActor` for work outside views. |
| [PreviewTraits.md](PreviewTraits.md) | Every preview builds and shows real data: a preview trait seeds an in-memory SwiftData container from bundled JSON, and views read it with `@Query`. |

One rule runs through all of them: never force-unwrap an optional.

## Using them

Clone this repo into the folder you run the agent from, then point the agent at it from the project's `CLAUDE.md` (Claude Code) or `AGENTS.md` (Codex and other agents). To apply the rules to every project, reference the clone from `~/.claude/CLAUDE.md` instead. The core of the instruction block I use:

```markdown
## Setup Instructions
Always read the markdown files in the skills folder. These are the Swift
coding conventions I follow, and I want you to follow them too.

## Architecture
- **Frontend**: SwiftUI
- **Backend**: Swift
- **Database**: SwiftData
- **Testing**: Swift Testing (not XCTest)

## Coding Standards
- Follow the existing code structure and patterns
- Write idiomatic Swift, not Swift in the style of another language
- Never force-unwrap an optional

## Testing Requirements
- Write tests for model code but not UI code
```

Despite the repo name, these are plain Markdown files rather than packaged agent skills, so any agent that reads an instruction file can use them.
