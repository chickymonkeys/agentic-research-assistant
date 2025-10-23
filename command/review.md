---
description: Reviews the last commit made and determines if the plan was executed completely, and documents any drift that occurred during implementation. Provide a plan file in the arguments for the review to analyze. It is strongly advised to run this command within the session of a plan execution, after running commit.
---

# Review Plan

You are tasked with validating that an implementation plan was correctly executed, verifying all success criteria and identifying any deviations or issues.

You will be given instructions, followed by a review that will contain user specific instructions and the plan file related to this implementation.

## Validation Process

### Step 1: Context Discovery

1. **Read the implementation plan** completely
2. **Identify what should have changed**:
   - List all files that should be modified
   - Note all success criteria (automated and manual)
   - Identify key functionality to verify

3. **Spawn parallel research tasks** to discover implementation:
   ```
   Task 1 - Verify pipeline/build:
   Run the data pipeline or build step and confirm outputs exist at expected paths.
   Check: exit status, produced artifacts in data/processed and output/
   Return: What was produced vs what the plan specified

   Task 2 - Verify code changes:
   Find all modified scripts/notebooks related to [feature].
   Compare variable constructions, joins/keys, CRS handling, and paths to the plan specifications.
   Return: File-by-file comparison of planned vs actual

   Task 3 - Verify tests/validation:
   Check if tests/validations were added or updated as specified.
   Run test scripts and any schema checks; capture results.
   Return: Test status and any missing coverage
   ```

### Step 2: Systematic Validation

For each phase in the plan:

1. **Check completion status**:
   - Look for checkmarks in the plan (- [x])
   - Verify the actual code matches claimed completion

2. **Run automated verification**:
   - Execute each command from "Automated Verification"
   - Document pass/fail status
   - If failures, investigate root cause

3. **Assess manual criteria**:
   - List what needs manual testing based on the plan's success criteria
   - Provide clear, specific steps for user verification
   - Adapt validation approach to the ticket scope
   - Emphasize checks that require domain expertise and human judgment

4. **Think deeply about edge cases**:
   - Were error conditions handled?
   - Are there missing validations?
   - Could the implementation break existing functionality?

### Step 3: Generate Validation Report

Create comprehensive validation summary and write it to the `thoughts/reviews` directory with a filename that matches the plan being reviewed (e.g., if reviewing `plan-feature-x.md`, save as `thoughts/reviews/feature-x-review.md`).

### Step 4: Update ticket status to 'reviewed' by editing the ticket file's frontmatter.

Use the todowrite tool to create a structured task list for the 4 steps above, marking each as pending initially.

```markdown
## Validation Report: [Plan Name]

### Implementation Status
✓ Phase 1: [Name] - Fully implemented
✓ Phase 2: [Name] - Fully implemented
⚠️ Phase 3: [Name] - Partially implemented (see issues)

### Automated Verification Results
✓ Pipeline runs successfully: `Rscript -e "targets::tar_make()"` / `make data`
✓ Tests pass: `Rscript -e "testthat::test_dir('tests')"` / `pytest -q`
⚠ Lint/format warnings: `lintr/styler` / `ruff/black --check` (3 warnings)

### Code Review Findings

#### Matches Plan:
- Variable construction and transformations match specification
- Join keys and CRS/projection handling are implemented as planned
- Validation checks align with plan

#### Deviations from Plan:
- Check the plan's "## Deviations from Plan" section (if present)
- For each deviation noted:
  - **Phase [N]**: [Original plan vs actual implementation]
  - **Assessment**: [Is the deviation justified? Impact on success criteria?]
  - **Recommendation**: [Any follow-up needed?]
- Additional deviations found during review:
  - Used different variable names in [file:line]
  - Added extra validation in [file:line] (improvement)

#### Potential Issues:
- Large join may be memory-intensive; consider chunking or indexing
- Missing seed may reduce determinism of outputs

### Manual Testing Required

Review plan-specific success criteria and verify implementation quality:

1. Outputs & Correctness:
   - [ ] Visually inspect figures and tables for accuracy and formatting
   - [ ] Confirm key statistics, sample sizes, and estimates within expected tolerances
   - [ ] Verify output files saved to expected paths with correct naming

2. Domain-Specific Validation (adapt based on ticket scope):
   - [ ] Data quality: distributions, outliers, missing patterns, logical consistency
   - [ ] Methodological validation: model diagnostics, specification tests, instrument strength (as applicable)
   - [ ] Geospatial accuracy: CRS/projections, spatial joins, boundary alignment (as applicable)
   - [ ] [Other checks relevant to this specific implementation]

3. Reproducibility:
   - [ ] Re-running pipeline produces identical outputs
   - [ ] Random operations use set seeds
   - [ ] Dependencies pinned in `renv.lock`/`requirements.txt`

4. Integration & Performance:
   - [ ] Compatible with upstream data sources and downstream analysis scripts
   - [ ] Performance acceptable with realistically-sized datasets
   - [ ] Documentation explains key methodological choices for maintainers

### Recommendations:
- Address linting warnings before merge
- Consider adding integration test for [scenario]
- Document new API endpoints
```

## Working with Existing Context

- Review the conversation history
- Check your todo list for what was completed
- Focus validation on work done in this session
- Be honest about any shortcuts or incomplete items

## Important Guidelines

1. **Be thorough but practical** - Focus on what matters
2. **Run all automated checks** - Don't skip verification commands
3. **Document everything** - Both successes and issues
4. **Think critically** - Question if the implementation truly solves the problem
5. **Consider maintenance** - Will this be maintainable long-term?
6. **Do not use task subagents** - All review work should be done exclusively in the main context to maintain consistency and avoid fragmentation

## Validation Checklist

Always verify:
- [ ] All phases marked complete are actually done
- [ ] Automated tests pass
- [ ] Code follows existing patterns
- [ ] No regressions introduced
- [ ] Error handling is robust
- [ ] Documentation updated if needed
- [ ] Manual test steps are clear

The validation works best after commits are made, as it can analyze the git history to understand what was implemented.

Remember: Good validation catches issues before they reach production. Be constructive but thorough in identifying gaps or improvements.

**review**

$ARGUMENTS

