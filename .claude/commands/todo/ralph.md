/ralph-loop:ralph-loop "Execute Multi-Session Implementation.

# Context & Instructions
1. **Count Sessions:** Count the number of files in `.claude/sessions`. Let this number be N.
2. **Sequential Processing:** You must process each session file one by one, in alphabetical/numerical order.
3. **Loop Control:** You are running in a Ralph Loop. Do not output the completion promise until ALL N sessions have been processed and verified.

# Workflow per Session
- Read session file `i`.
- Apply changes described in that session.

- **Mandatory Test:** Run `npm test` or `npx cypress run`. 
- If tests fail: Fix the code within the same iteration.
- If tests pass: Move to session `i+1` in the next iteration.

# Success Criteria
- All N sessions from `.claude/sessions` are implemented.
- **Zero Regressions:** All Cypress tests pass for the final state.
- Documentation updated.
- No linter errors.

Output <promise>COMPLETE</promise> ONLY after processing the last session and passing all tests." --max-iterations 100 --completion-promise "COMPLETE"