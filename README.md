# AFK

A [Claude Code](https://claude.com/claude-code) skill that lets you step away while the session keeps working on its own.

Before you leave, it interviews you about how to handle foreseeable problems (failing tests, ambiguity, blocked permissions, risky actions, hung jobs, low resources). Your answers are saved as a contract. It then runs unattended within that contract and gives you a report when you return.

## Install

```
git clone https://github.com/badhan-kv/AFK-skill.git
cp -r AFK-skill/AFK ~/.claude/skills/
```

Restart Claude Code (or start a new session) so the skill is picked up.

## Usage

```
/AFK
/AFK <task prompt>
```

Typically you run it mid-task, for example with a long training run already going. The skill infers the task and current state from the conversation and the system, plays it back in a few lines, and asks only about what is missing.

## How it works

1. **Intake**: infers the task and state; asks only about gaps (done criteria, time away, how to reach you).
2. **Scenario interview**: asks what to do for each relevant situation (failures, permissions, risky or irreversible actions, hangs, resources, unknowns). Anything risky that you do not approve is forbidden.
3. **Contract**: written to `.claude/afk/<timestamp>.md`; work starts only after you confirm.
4. **Autonomous run**: follows the contract, never waits for input, parks anything it cannot handle and continues with independent work, verifies each step, and notifies you on finish or block.
5. **Return report**: what was completed, decisions made on your behalf, parked items, and next steps.

## Notes

- The skill cannot grant permissions. A permission prompt with nobody present stalls the session, so set your permission mode or allowlist before leaving.
- Keep the machine awake and the session alive (e.g. run inside tmux) while away.

## License

MIT
