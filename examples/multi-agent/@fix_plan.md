# Fix Plan — Multi-Agent Example

## [researcher] Phase 1: Research
- [ ] Research [TOPIC] and gather relevant information
- [ ] Identify key concepts, patterns, and best practices
- [ ] Save findings to `docs/research/[topic].md`

## [developer] Phase 2: Document Generation
- [ ] Read research from `docs/research/[topic].md`
- [ ] Generate output document at `output/[topic]_report.md`
- [ ] Verify the output is complete and well-structured

## [reviewer] Phase 3: Review & Publish
- [ ] Review the generated document in `output/[topic]_report.md`
- [ ] Apply corrections or improvements if needed
- [ ] Save final version and record completion in `output/completed.txt`

## Notes
- Replace `[TOPIC]` and `[topic]` with your actual topic before running
- Replace `researcher`, `developer`, `reviewer` with your agents from `~/.copilot/agents/`
- Ralph detects the active agent from the first section with incomplete `[ ]` tasks
