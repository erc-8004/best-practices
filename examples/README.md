# ERC-8004 Agent Examples

This directory contains example registration files for different types of AI agents, demonstrating how to apply the ERC-8004 standard across various use cases.

## Examples

### 1. **Simple Chatbot** (`chatbot-agent.json`)
A basic customer support agent with natural language capabilities.

**Use Case:** Customer service, FAQ assistance, general inquiries
**Key Features:**
- Text conversation
- Question answering
- Sentiment analysis
- Multi-language support

### 2. **DeFi Trading Agent** (`defi-agent.json`)
An autonomous agent for cryptocurrency trading and portfolio management.

**Use Case:** Automated trading, yield farming, portfolio rebalancing
**Key Features:**
- Market analysis
- Trade execution
- Risk management
- Real-time price monitoring

### 3. **Data Analyst Agent** (`data-analyst-agent.json`)
A business intelligence agent that analyzes data and generates insights.

**Use Case:** Business analytics, data visualization, reporting
**Key Features:**
- Data analysis
- Chart generation
- Report creation
- Predictive analytics

### 4. **Creative Agent** (`creative-agent.json`)
A multimodal agent for generating creative content (images, music, text).

**Use Case:** Content creation, marketing materials, artistic projects
**Key Features:**
- Image generation
- Music composition
- Text-to-image/speech
- Style transfer

## Structure

Each example follows the ERC-8004 registration format with:

- **Basic Info**: Name, description, image
- **Endpoints**: Communication channels (MCP, A2A, wallet, etc.)
- **Capabilities**: OASF skills and domains
- **Registration**: On-chain verification details
- **Production Ready**: Active status and x402 payment support

## Using These Examples

1. **Choose an example** that matches your agent type
2. **Customize** the fields with your agent's specific details
3. **Update endpoints** with your actual API URLs
4. **Select appropriate** OASF skills and domains from:
   - `../all_skills.json`
   - `../all_domains.json`
5. **Register** your agent on-chain and get the `agentId`
6. **Deploy** your metadata file to a public URL

## Validation

Before deploying, ensure your metadata:
- ✅ Follows valid JSON syntax
- ✅ Uses correct OASF skills/domains
- ✅ Has accessible endpoints
- ✅ Includes on-chain registration details
- ✅ Has proper image URLs

## Resources

- [ERC-8004 Specification](https://eips.ethereum.org/EIPS/eip-8004)
- [OASF Schema v0.8.0](https://github.com/agntcy/oasf/)
- [Main Best Practices Guide](../README.md)
