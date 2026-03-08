# Lessons Learned

- Run A and B serially unless concurrency is itself the variable under test.
- Prefer absolute paths for subagent-visible artifacts.
- Treat harness failures as invalid runs, not poor model performance.
- Source-grounded cloze tests often measure recall more than understanding.
- Objective MCQ can still saturate on strong models.
- Explanation rubrics can also saturate if too coarse.
- The benchmark must target the capability the tested artifact is supposed to improve.
- Use blind judging for explanation quality.
- Record failures and benchmark-version changes in a lab notebook.
