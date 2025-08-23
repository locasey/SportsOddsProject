## Workflow
1) Create a branch: feat/<ticket> or fix/<ticket>.
2) Read AGENT_SCOPE.json and only touch allowed areas.
3) Run `npm run test && npm run lint && npm run typecheck`.
4) Open PR using the template.

## Commit style
type(scope): summary
e.g., feat(chatbot): add profit-expectation handler
