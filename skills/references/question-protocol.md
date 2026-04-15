# Question Protocol

Structured brainstorming protocol for PRD creation. Follow these phases and
rules to guide the conversation from idea to document.

## Phases

### 1. Discovery

Gather initial context about the idea or problem space.

- what is the core problem or opportunity
- who are the affected users
- what prompted this initiative

### 2. Understanding

Deepen knowledge of requirements and constraints.

- WHAT specific features do users need
- WHY does this provide business value
- WHO are the target users and what are their current workflows
- what are the success criteria
- what are the known constraints

### 3. Options

Present product approaches for the user to evaluate.

- offer 2-3 distinct approaches with clear trade-offs
- lead with the recommended approach and explain why
- each approach should differ meaningfully in scope, phasing, or strategy
- wait for the user or clarification-policy outcome before proceeding

### 4. Refinement

Refine the selected approach with targeted follow-ups.

- clarify scope boundaries for the chosen approach
- confirm phasing and feature priority
- validate success criteria and metrics
- resolve any remaining open questions

### 5. Creation

Generate the PRD document using the gathered context.

- read and fill the PRD template
- every section should reflect confirmed decisions
- unresolved items go into Open Questions

## Rules

### Interactive Question Enforcement

- every question should use a blocking question tool when the runtime provides
  one
- if no such tool is available and clarification policy is `human_required`,
  ask the question as the complete reply and stop
- if clarification policy allows AI resolution, record the AI-made answer in
  `_decisions.md` instead of silently continuing

### Question Limits

- ask only one question per message
- use multiple-choice questions whenever bounded options can be offered
- use open-ended questions only when the answer space is genuinely unbounded
- include an `Other — describe` fallback when helpful
- complete at least one full clarification round before presenting options
- target 3-6 meaningful clarification questions unless update mode or existing
  approved decisions already make fewer necessary

### Progression Gates

- do not present approaches until purpose, constraints, affected actors, and
  success criteria are materially clear
- do not draft the PRD until an approach is selected, approved, or explicitly
  auto-decided under the active clarification policy

### Focus Boundaries

Questions must focus on:

- WHAT
- WHY
- WHO
- business value
- user or operator experience
- constraints and success criteria

Never ask implementation questions about:

- databases
- APIs
- code structure
- frameworks
- testing strategies
- architecture patterns
- deployment topology

### YAGNI Principle

- ruthlessly remove non-essential features during refinement
- challenge every feature against MVP necessity
- prefer smaller, well-defined scope over breadth

### Simplicity Trap

Do not skip the question protocol because the feature appears simple. Simple
features often hide the most dangerous product assumptions.
