# PR Writing Regression Cases

Run the cases in `evals.json` with the supplied Markdown fixtures. They cover title priority,
plain wording, review structure, feature framing, release-type inference, title-only scope,
and template precedence.
All repository names, changes, and validation facts in the fixtures are illustrative.

## Execution

Use a fresh agent context for each case and skill version. Supply the selected `SKILL.md`,
the case's prompt, and the files listed for that case. Make that version's own `references/`
available for the agent to read on demand. Keep fixture scenarios distinct from those examples.
Treat the fixture as the complete repository evidence. Keep these writing-only runs offline:
do not inspect the real checkout or run GitHub, git mutation, commit, or publishing commands.

For a version comparison, run the same cases against the previous and revised skill. Save
the title, description, any separate validation report, and the execution transcript outside
the tracked skill directory. Keep model and settings the same between versions.

## Assessment

Check each case's expectations against both the output and execution transcript. Assess title
priority and readability with a human or model reviewer; keyword checks alone cannot prove
either. Check concrete constraints, required sections, and requested-field scope separately.
Distinguish a prose review from proof that publishing commands or GitHub integration work.

The no-publication checks verify harness compliance, not a difference between skill versions.
These are regression cases, not a statistically reliable estimate of reduced user steering.
Compare several runs or models before making broader claims about reliability.
