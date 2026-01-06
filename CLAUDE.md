# CLAUDE.md

This document serves as institutional memory for the Arkade documentation. It trains Claude Code on our standards, patterns, and practices to ensure consistent, high-quality documentation contributions.

## Working Relationship

### Transparency and Honesty
- ALWAYS ask for clarification rather than making assumptions about Arkade's technical implementation or Bitcoin concepts
- NEVER lie, guess, or make up information about how Arkade works, especially regarding security, finality, or settlement mechanisms
- If uncertain about Bitcoin, UTXO, or Arkade-specific terminology, ask for clarification before proceeding
- Surface concerns immediately if documentation appears technically inaccurate or could mislead developers

### Communication Style
- Use second-person perspective ("you" not "we") when addressing readers
- Write concisely and technically - developers value precision over prose
- Default to showing code examples before lengthy explanations
- When explaining complex concepts (VTXOs, batch outputs, virtual mempool), use progressive disclosure: start simple, then layer in complexity

## Project Context

### Technology Stack
- **Documentation Framework**: Mintlify with MDX files
- **Configuration**: docs.json defines navigation, theming, and redirects
- **Content Format**: MDX files with YAML frontmatter
- **Components**: Mintlify-specific React components (Card, CardGroup, Accordion, Frame, etc.)
- **Development**: Local preview via `mintlify dev` command

### Project Structure
```
docs/
├── docs.json                 # Main configuration
├── index.mdx                 # Homepage
├── primer.mdx                # Technical primer
├── roadmap.mdx               # Roadmap
├── learn/                    # Core concepts and theory
│   ├── glossary.mdx
│   ├── pillars/              # Arkade architecture fundamentals
│   ├── security/             # Security model and guarantees
│   └── faq/                  # Frequently asked questions
├── wallets/                  # Wallet SDK documentation
│   └── v0.3/                 # Versioned wallet docs
├── contracts/                # Smart contract guides
├── arkd/                     # Server & API reference
│   ├── txs/                  # Transaction workflows
│   ├── core-services/        # API layer documentation
│   ├── components/           # Technical components
│   └── server-security/      # Security mechanisms
└── experimental/             # Experimental features
    └── arkade-language/      # Arkade Script documentation
```

### Core Technologies Referenced
- **Bitcoin**: Base layer blockchain, UTXO model, scripting
- **TypeScript/JavaScript**: Primary SDK language
- **React Native**: Mobile wallet support
- **Node.js**: Server-side SDK
- **Rust & Go**: Alternative SDK implementations

## Content Strategy

### Documentation Philosophy
- **Just-in-time documentation**: Document enough for developer success, avoid redundancy
- **Code-first approach**: Show working examples before explaining theory
- **Progressive disclosure**: Start with simple use cases, introduce complexity as needed
- **Trust but verify**: Reference TEE-based security, verifiable execution, and on-chain exit paths

### Target Audience
Primary: Bitcoin developers building wallets, payments, or financial applications
Secondary: Protocol researchers understanding Arkade's architecture
Tertiary: End users learning about Bitcoin self-custody and programmability

### Content Types

#### Learn Tab
Conceptual documentation explaining Arkade's architecture:
- **Pillars**: Core architectural components (VTXOs, batch outputs, virtual mempool)
- **Security**: Trust model, unilateral exit, finality guarantees
- **FAQ**: Common questions about Arkade vs Ark, Bitcoin compatibility, operator role

#### Wallet & Payments Tab
Practical SDK guides with code examples:
- Setup and initialization
- Payment operations (send, receive, balance)
- Advanced features (VTXO management, storage adapters)
- Platform-specific guides (React Native, Service Workers)

#### Contracts & Apps Tab
Smart contract development guides:
- Arkade Script overview
- Simple contracts (Lightning swaps, escrow, DLCs)
- UTXO-based contract patterns

#### Server & API Reference Tab
Backend and infrastructure documentation:
- Arkd server overview
- Transaction workflows (boarding, execution, settlement, exiting)
- API services (Ark service, Indexer service)
- System components (intent system, scheduled sessions)

#### Experimental Tab
Cutting-edge features under development:
- Arkade language specification
- Advanced contract patterns (AMMs, prediction markets, synthetic assets)
- Clearly marked as experimental to set expectations

## Documentation Requirements

### Frontmatter Standards
Every MDX file MUST include YAML frontmatter with:
```yaml
---
title: "Clear, Descriptive Title"
description: "Concise summary for SEO and navigation (under 160 chars)"
icon: "relevant-icon-name"  # Optional but recommended
---
```

### Writing Conventions

#### Voice and Tone
- Use second-person ("you can build", not "we can build")
- Be direct and technical, avoid marketing language
- Present tense for describing current functionality
- Future tense only for explicitly roadmapped features

#### Bitcoin and Arkade Terminology
Use precise technical terms:
- **VTXO** (Virtual TXO): Offchain Bitcoin outputs with unilateral exit rights
- **Batch output**: Compressed representation of multiple VTXOs in a single on-chain output
- **Virtual mempool**: Offchain execution environment for parallel VTXO transactions
- **Batch swap**: Onchain settlement transaction coordinating multiple VTXO state transitions
- **Connector output**: Bitcoin output linking batches and enabling state transitions
- **Arkade Script**: Stateful, expressive smart contract language compiling to Bitcoin execution flows
- **TEE** (Trusted Execution Environment): Hardware-enforced security for verifiable execution
- **Unilateral exit**: User's ability to withdraw funds without operator cooperation

Distinguish clearly:
- **Ark Protocol**: Foundation providing VTXOs, batch outputs, batch swaps
- **Arkade**: Execution layer extending Ark with virtual mempool, smart contracts, programmability

#### Code Blocks
ALWAYS specify language for syntax highlighting:
```typescript
// Good
import { Wallet } from '@arkade-os/sdk';
```

```
// Bad - missing language tag
import { Wallet } from '@arkade-os/sdk';
```

Prefer realistic, runnable code examples over pseudocode.

#### Links
- Use relative paths for internal links: `[VTXOs](/learn/pillars/vtxos)`
- NEVER use absolute URLs for internal documentation
- External links should use full URLs: `[GitHub](https://github.com/arkade-os)`
- Link to relevant context liberally - help developers discover related concepts

#### Images
- Always include descriptive alt text: `![Arkade transaction flow diagram](/images/flow.png)`
- Store images in `/images/` directory
- Use PNG for diagrams, JPG for photos, SVG for logos when possible

#### Mintlify Components
Use built-in components for enhanced presentation:
- `<Card>` and `<CardGroup>`: Link collections
- `<Accordion>` and `<AccordionGroup>`: Collapsible content
- `<Frame>`: Image containers with styling
- `<CodeGroup>`: Multi-language code examples
- `<Tabs>`: Alternative content presentation

Example:
```mdx
<CardGroup cols={2}>
  <Card title="Setup Guide" icon="rocket" href="/wallets/v0.3/setup">
    Get started with Arkade wallets
  </Card>
  <Card title="API Reference" icon="code" href="/arkd/core-services/overview">
    Explore the API layer
  </Card>
</CardGroup>
```

### Structure Patterns

#### Getting Started Pages
1. Brief introduction (1-2 paragraphs)
2. Prerequisites (if any)
3. Installation/setup code
4. Quick example
5. Next steps with navigation cards

#### Concept Pages
1. Clear definition
2. Why it matters (problem it solves)
3. How it works (technical explanation)
4. Visual diagrams when helpful
5. Code examples showing practical usage
6. Related concepts

#### API Reference Pages
1. Overview of the service/component
2. Key methods/endpoints
3. Request/response examples
4. Error handling
5. Integration patterns

## Git Workflow

### Branch Strategy
- Main branch: `master`
- Create feature branches for significant documentation additions or restructuring
- Use descriptive branch names: `docs/vtxo-explainer`, `fix/broken-links`

### Commit Practices
- Write clear commit messages describing the documentation change
- NEVER use `--no-verify` when committing
- Use conventional commit style: `docs: add VTXO lifecycle diagram`
- Keep commits focused on single logical changes

### Pull Requests
- Reference related issues or discussions
- Provide context for why documentation is being added/changed
- Request review from technical experts for complex protocol explanations

## Prohibited Practices

### Content
- ❌ DO NOT omit required frontmatter (title, description)
- ❌ DO NOT use absolute URLs for internal links
- ❌ DO NOT forget language tags on code blocks
- ❌ DO NOT make claims about Bitcoin protocol changes - Arkade requires none
- ❌ DO NOT conflate Arkade with Ark protocol - they're related but distinct
- ❌ DO NOT oversimplify security model - be precise about trust assumptions
- ❌ DO NOT promise features not yet implemented unless clearly marked as roadmap/experimental

### Structure
- ❌ DO NOT create orphaned pages (pages not linked from docs.json navigation)
- ❌ DO NOT duplicate content - link to existing explanations
- ❌ DO NOT break existing URLs without adding redirects in docs.json
- ❌ DO NOT nest navigation more than 3 levels deep

### Style
- ❌ DO NOT use first-person plural ("we", "our product")
- ❌ DO NOT use marketing language or superlatives without technical backing
- ❌ DO NOT use vague terms like "soon", "later", "eventually" - be specific or omit
- ❌ DO NOT anthropomorphize code ("the wallet wants to", "VTXOs think")

## Common Documentation Patterns

### Versioning
When documenting versioned features (like wallets), maintain version-specific directories:
```
wallets/
└── v0.3/
    ├── introduction.mdx
    ├── setup.mdx
    └── ...
```

Add redirects in docs.json when URLs change between versions:
```json
{
  "source": "/wallets/setup",
  "destination": "/wallets/v0.3/setup"
}
```

### Cross-referencing
Link generously to related concepts:
- From implementation guides to conceptual explanations
- From FAQ answers to detailed documentation
- From simple examples to advanced patterns

Example: When explaining payments, link to [VTXOs](/learn/pillars/vtxos), [virtual mempool](/learn/pillars/virtual-mempool), and [settlement](/wallets/v0.3/settlement).

### Code Examples
Follow this pattern:
1. Show the simplest working example first
2. Explain what it does and why
3. Show variations for common use cases
4. Link to advanced patterns or API docs for depth

### Navigation Updates
When adding new pages, update docs.json navigation:
```json
{
  "group": "Group Name",
  "pages": [
    "path/to/new-page",
    "path/to/another-page"
  ]
}
```

Maintain logical grouping and ordering that matches developer learning paths.

## Quality Standards

### Before Submitting
- [ ] Frontmatter includes title, description, and icon (if applicable)
- [ ] All code blocks have language tags
- [ ] Internal links use relative paths
- [ ] External links are valid and use HTTPS
- [ ] Images have descriptive alt text
- [ ] Technical accuracy verified for Bitcoin and Arkade concepts
- [ ] No marketing language or unsubstantiated claims
- [ ] Navigation updated in docs.json if needed
- [ ] Redirects added if URLs changed
- [ ] Local preview tested with `mintlify dev`

### Maintenance
- Review documentation quarterly for accuracy as Arkade evolves
- Update code examples when SDKs change
- Mark experimental features that graduate to stable
- Archive or redirect deprecated documentation
- Keep README.md updated with contributor instructions

## Additional Resources

- [Arkade GitHub Organization](https://github.com/arkade-os)
- [TypeScript SDK](https://github.com/arkade-os/ts-sdk)
- [Reference Wallet](https://github.com/arkade-os/wallet)
- [Mintlify Documentation](https://mintlify.com/docs)
- [Mintlify Components Reference](https://mintlify.com/docs/components)

## Questions?

When in doubt:
1. Check existing documentation for established patterns
2. Ask the Arkade team in Telegram: https://t.me/arkade_os
3. Reference Bitcoin technical documentation for base layer concepts
4. Consult Mintlify docs for MDX and component usage

This document evolves with the project. When you notice gaps or improvements, propose changes to keep it current and useful.
