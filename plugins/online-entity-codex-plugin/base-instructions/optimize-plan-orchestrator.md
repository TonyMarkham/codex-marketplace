# Optimize Plan Orchestrator

Orchestrate bounded iterative plan optimization. Do not edit the plan directly and do not act as a
vote counter.

Use `$optimize-plan-orchestrator` and follow its full contract. In particular:

- Build an immutable directive packet from explicit user decisions and applicable repository
  instructions. User decisions outrank repo defaults; repo directives outrank reviewer preference.
- Send that packet, mode, paths, and one neutral audit lens to every fresh worker. Never send pass
  history, findings, outcomes, clean state, or suppression instructions.
- Cycle the three configured lenses. Each worker still performs all mandatory gates.
- Count a pass clean only when its directive ledger, complete coverage, exact evidence, unverified
  list, materiality classification, and no-edit result support the claim.
- Treat `AGENTS.md` coding and organization directives as material. Ignore prose polish, optional
  hardening, personal taste, and speculative robustness without material evidence.
- Inspect the actual diff and arbitrate each result. Reject any edit that overrides a user decision
  or binding repo directive.
- Resolve apparent flip-flops by authority and operational behavior. Refinement or corrected
  rationale is not oscillation. Stop for flip-flop only when material evidence-backed reversals are
  genuinely irreconcilable.
- Stop after three consecutive clean distinct lenses. Do not run extra confidence or polish passes.
- Distinguish evidence-audited convergence from actual compile validation.

If the user changes a directive mid-run, interrupt the worker, rebuild the packet, reset the clean
streak, and continue fresh. Single-agent fallback must never be reported as blind convergence.
