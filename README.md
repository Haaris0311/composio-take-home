# Composio App Research - AI Product Ops Intern Assignment

## Overview

An automated multi-agent research pipeline that analyzed **100 apps across 10 categories** to determine their suitability as Composio agent toolkits. The system evaluates authentication methods, self-serve availability, API surface, and buildability.

## Key Findings

- **74/100 apps are ready for agent toolkit integration today**
- OAuth2 dominates (48%), followed by API keys (40%)
- 54% offer self-serve free access for developers
- Developer/Productivity tools have zero friction; Finance/AI categories are the most gated
- The #1 blocker is "no API exists" (6 apps), not technical complexity

## Architecture

```
┌─────────────────────────────────────────────────┐
│           Claude Agent SDK Orchestrator           │
├─────────────────────────────────────────────────┤
│                                                   │
│  Step 1: Extract 100 apps from Notion page        │
│          (via loadPageChunk API)                  │
│                                                   │
│  Step 2: Parallel Research (5 agents x 20 apps)   │
│          ┌──────┐ ┌──────┐ ┌──────┐ ...         │
│          │Batch1│ │Batch2│ │Batch3│              │
│          └──────┘ └──────┘ └──────┘              │
│                                                   │
│  Step 3: Structured JSON Output (schema-enforced) │
│                                                   │
│  Step 4: Verification Agent (20-app sample)       │
│                                                   │
│  Step 5: Corrections + Final HTML generation      │
│                                                   │
└─────────────────────────────────────────────────┘
```

## How to Run

### Prerequisites
- Python 3.10+
- Anthropic API key (for Claude)

### Quick Start

```bash
# Clone the repo
git clone <repo-url>
cd composio-assignment

# Install dependencies
pip install anthropic aiohttp

# Run the research agent
python src/research_agent.py --input apps_list.json --output results.json

# Open the HTML report
open index.html
```

### Using Claude Agent SDK (as built)

The actual research was conducted using Claude Agent SDK's workflow system:

```javascript
// 5 parallel research agents
const results = await parallel(batches.map((batch, i) => () => {
  return agent(PROMPT_TEMPLATE(batch), {
    schema: RESEARCH_SCHEMA,
    model: 'sonnet'
  })
}))

// Independent verification agent
const verification = await agent(VERIFY_PROMPT, {
  schema: VERIFY_SCHEMA
})
```

## Project Structure

```
composio-assignment/
├── index.html              # Final deliverable (self-contained HTML page)
├── README.md               # This file
├── apps_list.json          # Input: 100 apps extracted from Notion
├── research_results.json   # Output: full research data (100 apps)
├── apps_data.json          # Compact data for HTML embedding
└── src/
    └── research_agent.py   # Python research agent (reference implementation)
```

## Verification

- **Method:** Stratified sample of 20 apps (2 per category)
- **First-pass accuracy:** 80% (16 correct, 4 uncertain due to newness)
- **After corrections:** 90%+ (zero outright factual errors found)
- **Common uncertainty:** Newer apps (Pylon, Twenty, Devin, systeme.io) with limited public API docs

## What the Agent Got Right

- Auth methods for well-known platforms (Salesforce, GitHub, Stripe, Slack)
- Self-serve classification for mainstream tools
- Buildability verdicts align with actual developer experience
- Identifying truly gated apps (PitchBook, DealCloud, Amazon SP)

## Where Humans Were Needed

1. **Notion page extraction** - Client-side rendering required custom API approach
2. **Niche app research** - fanbasis, higgsfield, Paygent have minimal public docs
3. **MCP server verification** - Rapidly evolving ecosystem
4. **Pattern interpretation** - Data to insight requires human judgment
5. **Verification sampling** - Chose stratified sample for fair coverage

## Time Breakdown

| Phase | Time | Method |
|-------|------|--------|
| Notion extraction | 5 min | Automated (API) |
| Research (100 apps) | ~70 sec | 5 parallel agents |
| Verification | ~2 min | 1 verification agent |
| HTML design & assembly | 30 min | Human + AI |
| Pattern analysis | 15 min | Human synthesis |
| **Total** | **~50 min** | Mostly automated |

## License

MIT
