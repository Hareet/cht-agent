# CHT Hierarchical Multi-Agent System: Technical POC Plan

## Notes
We created this design based off:
- our manual multi-agent setup that created [Historical Targets Context](https://github.com/Hareet/cht-core/blob/301ee2aadbe31a13faa5e0c086b0d25fcaf65022/webapp/src/ts/modules/analytics/CHW-Historical-Targets-Context.md) and [PR](https://github.com/Hareet/cht-core/pull/1)
- learnings from [Agentic Design Patterns](https://docs.google.com/document/d/1rsaK53T3Lg5KoGwvf8ukOUvbELRtH-V0LnOIFDxBryE/preview?pli=1&tab=t.0)
- This plan was generated and revised across days with Claude and edited/verified by me. 

## Summary

This document outlines a pragmatic proof-of-concept for a hierarchical multi-agent system designed to assist with CHT development workflows. The system leverages existing CHT testing infrastructure, employs context-based learning, and focuses on practical value delivery rather than theoretical completeness.

## System Architecture Overview

```
Master Supervisor (Orchestrator)
├── Research Supervisor
│   ├── Documentation Search Agent
│   └── Context Analysis Agent
├── Development Supervisor
│   ├── Code Generation Agent
│   └── Test Environment Agent
└── QA Supervisor
    ├── Code Validation Agent 
    └── Test Orchestration Agent
```

## Core Design Principles

1. **Tool Orchestration Over Reinvention**: Agents act as intelligent coordinators of existing CHT tools (cht-conf, cht-toolbox, cht-datasource, cht-docs, npm scripts) rather than implementing custom validation
2. **Context-Based Learning**: Lightweight file-based memory system that mirrors CHT repository structure
3. **Incremental Value Delivery**: Each agent provides immediate value while building knowledge over time
4. **Developer-Friendly Integration**: Seamless integration with existing npm scripts and development workflows

## Agent Specifications

### 1. Master Supervisor (Orchestrator)

**Responsibilities:**
- Route tasks to appropriate supervisors based on GitHub issue analysis
- Maintain global task queue and workflow state
- Coordinate cross-functional requirements
- Generate final consolidated reports

**Implementation:**
```typescript
export class MasterOrchestrator {
  private graph: CompiledStateGraph<typeof SystemState.State>;
  
  async processIssue(issueNumber: number): Promise<WorkflowResult> {
    const issue = await this.fetchGitHubIssue(issueNumber);
    const analysis = await this.analyzeIssue(issue);
    
    // Route to supervisors based on analysis
    const workflow = this.determineWorkflow(analysis);
    return await this.graph.invoke({
      issue,
      workflow,
      messages: []
    });
  }
}
```

### 2. Research Supervisor and Agents

**Documentation Search Agent:**
- Searches CHT documentation using existing search infrastructure
- Maintains context catalog of relevant documentation sections
- Cross-references with previous search results

**Context Analysis Agent:**
- Loads and analyzes relevant context files from previous resolutions
- Identifies patterns from similar past issues
- Provides historical insights to other agents

**Context File Structure:**
```typescript
interface ContextFile {
  id: string;
  issueNumber: number;
  techStack: string[];
  category: 'forms' | 'tasks' | 'permissions' | 'workflows';
  summary: string;
  codePatterns: CodePattern[];
  designChoices: DesignDecision[];
  resolution: ResolutionDetails;
  timestamp: Date;
}
```

### 3. Development Supervisor and Agents

**Code Generation Agent:**
- Generates code following CHT patterns from context files
- Uses Claude Sonnet 3.5 for implementation
- Adheres to CHT coding standards by default

**Test Environment Agent:**
- Orchestrates cht-conf commands to create test configurations
- Sets up test data using existing CHT tooling
- Manages k3d/Docker environments for integration testing

### 4. QA Supervisor and Agents

**Code Validation Agent:**
- Orchestrates linting and compiling (ESlint)
- Runs Shellcheck for shell scripts
- Parses and enriches linting results with context
- Validates CHT coding standards and styles were met
- Matches standard and style with examples from existing code in similar category 

**Test Orchestration Agent:**
- Intelligently selects test suites based on changes
- Runs tests through npm scripts
- Analyzes coverage reports from nyc
- Detects flaky tests and patterns

## Memory and Learning System

### Dual-Layer Context Architecture

The system implements both **shared context files** (for cross-agent knowledge) and **agent-specific context** (for specialized expertise).

### Shared Context Structure

```
agent-memory/
├── shared-contexts/
│   ├── webapp/
│   │   ├── src/
│   │   │   └── js/
│   │   │       ├── modules/
│   │   │       │   ├── forms/
│   │   │       │   ├── tasks/
│   │   │       │   ├── reports/
│   │   │       │   └── targets/
│   ├── api/
│   │   └── src/
│   │       └── controllers/
│   └── shared-libs/
│       ├── rules-engine/
│       ├── calendar-interval/
│       └── transitions/
├── agent-specific/
│   ├── qa-supervisor/
│   │   ├── test-failures/
│   │   │   ├── flaky-tests.json
│   │   │   └── error-patterns.json
│   │   ├── coverage-trends/
│   │   └── validation-rules/
│   ├── test-environment/
│   │   ├── cht-toolbox-patterns/
│   │   ├── cht-datasource-configs/
│   │   ├── cht-conf-recipes/
│   │   └── data-generation-templates/
│   ├── research/
│   │   ├── doc-references/
│   │   ├── solution-patterns/
│   │   └── api-usage-examples/
│   └── development/
│       ├── code-templates/
│       ├── refactoring-patterns/
│       └── migration-strategies/
└── indices/
    ├── by-category.json
    ├── by-component.json
    └── by-error-type.json
```

### Comprehensive Category System

```typescript
interface ContextFile {
  id: string;
  issueNumber: number;
  techStack: string[];
  
  // Expanded category system based on CHT architecture
  category: CHTCategory;
  subCategory?: string;
  
  // Core context
  summary: string;
  codePatterns: CodePattern[];
  designChoices: DesignDecision[];
  resolution: ResolutionDetails;
  
  // Agent-specific metadata
  agentContributions: {
    [agentId: string]: AgentContext;
  };
  
  timestamp: Date;
}

type CHTCategory = 
  // Core Application Components
  | 'forms'           // Data collection forms (contact, app forms)
  | 'tasks'           // Scheduled actions and reminders
  | 'targets'         // Performance metrics and goals
  | 'reports'         // Data aggregation and analytics
  | 'workflows'       // Multi-step processes and state machines
  
  // System Architecture
  | 'permissions'     // Roles and access control
  | 'messaging'       // SMS, alerts, notifications
  | 'sync'            // Replication and offline sync
  | 'purging'         // Data retention and cleanup
  
  // Configuration & Settings
  | 'app-settings'    // Application configuration
  | 'contact-hierarchy' // Organizational structure
  | 'translations'    // i18n and localization
  | 'branding'        // UI customization
  
  // Technical Components  
  | 'api'             // REST endpoints and controllers
  | 'database'        // CouchDB views and indexes
  | 'sentinel'        // Background processing and transitions
  | 'webapp'          // Frontend components and UI
  
  // Development Tools
  | 'testing'         // Test infrastructure and patterns
  | 'deployment'      // Docker, k8s, hosting configs
  | 'performance'     // Optimization and monitoring
  | 'migrations'      // Version upgrades and data migrations
  
  // CHT-Conf Specific
  | 'cht-conf'        // Configuration compilation
  | 'cht-datasource'  // Data pipeline configurations
  | 'cht-toolbox'     // Utility functions and helpers
  | 'test-data'       // Test data generation patterns;
```

### Agent-Specific Context Examples

```typescript
// QA Supervisor Context
interface QAContext {
  testFailures: {
    testPath: string;
    failureCount: number;
    lastFailure: Date;
    errorPatterns: string[];
    flakyScore: number; // 0-1, probability of being flaky
    environments: string[]; // Where it failed
  }[];
  
  coverageHistory: {
    component: string;
    trends: { date: Date; coverage: number }[];
    uncoveredCriticalPaths: string[];
  }[];
  
  validationPatterns: {
    rule: string;
    frequency: number;
    autoFixAvailable: boolean;
    commonViolations: string[];
  }[];
}

// Test Environment Agent Context  
interface TestEnvContext {
  chtToolboxUsage: {
    function: string;
    useCase: string;
    parameters: any;
    successRate: number;
  }[];
  
  chtDatasourceConfigs: {
    dataType: string;
    generator: string;
    config: any;
    validationRules: string[];
  }[];
  
  chtConfPatterns: {
    command: string;
    scenario: string;
    flags: string[];
    troubleshooting: string[];
  }[];
  
  dataGenerationTemplates: {
    entity: string; // 'patient', 'chw', 'facility'
    template: any;
    relationships: string[];
    constraints: string[];
  }[];
}

// Research Agent Context
interface ResearchContext {
  documentationMap: {
    topic: string;
    urls: string[];
    relevantSections: string[];
    codeExamples: string[];
    lastUpdated: Date;
  }[];
  
  solutionPatterns: {
    problem: string;
    solutions: {
      approach: string;
      implementation: string;
      tradeoffs: string[];
      successRate: number;
    }[];
  }[];
  
  apiUsageExamples: {
    endpoint: string;
    method: string;
    useCase: string;
    parameters: any;
    response: any;
    errorHandling: string[];
  }[];
}
```

### Category-Based Context Retrieval

```typescript
// Context retrieval with comprehensive categories
class ContextRetrieval {
  async findRelevantContext(query: ResearchQuery): Promise<ContextFile[]> {
    const contexts: ContextFile[] = [];
    
    // Primary category matching
    const primaryMatches = await this.searchByCategory(query.category);
    
    // Cross-category relationships
    const relatedCategories = this.getRelatedCategories(query.category);
    // Example: 'forms' relates to 'validation', 'permissions', 'workflows'
    
    for (const related of relatedCategories) {
      const relatedContexts = await this.searchByCategory(related);
      contexts.push(...relatedContexts);
    }
    
    // Component-specific search
    if (query.component) {
      const componentContexts = await this.searchByComponent(query.component);
      contexts.push(...componentContexts);
    }
    
    // Agent-specific enrichment
    const enrichedContexts = await this.enrichWithAgentContext(contexts, query.agentId);
    
    return this.rankByRelevance(enrichedContexts, query);
  }

  private getRelatedCategories(category: CHTCategory): CHTCategory[] {
    const relationships = {
      'forms': ['workflows', 'permissions', 'validation', 'reports'],
      'tasks': ['targets', 'workflows', 'messaging'],
      'permissions': ['forms', 'reports', 'api'],
      'sync': ['database', 'api', 'performance'],
      'cht-conf': ['app-settings', 'contact-hierarchy', 'forms', 'tasks'],
      'testing': ['cht-conf', 'test-data', 'performance'],
      // ... more relationships
    };
    
    return relationships[category] || [];
  }
}
```

### Benefits of Agent-Specific Context

1. **Specialized Knowledge Accumulation**
   - QA agents build expertise in test patterns without cluttering general context
   - Test Environment agents master tool-specific configurations
   - Each agent becomes more effective in its domain

2. **Faster Retrieval**
   - Agent-specific indices reduce search space
   - Relevant context retrieved based on agent role
   - No need to filter out unrelated information

3. **Better Pattern Recognition**
   - Flaky test patterns emerge from QA-specific data
   - Data generation patterns identified by Test Environment agent
   - Solution patterns discovered by Research agent

4. **Improved Recommendations**
   - Context-aware suggestions based on agent expertise
   - Historical success rates inform decision making
   - Cross-agent insights through shared context layer

## Integration with CHT Infrastructure
This section got over-engineered and we'll plan ideas later. Saving the Development Workflow Integration as an example.

### Development Workflow Integration

1. **Issue Created**: Template has keywords, schema to allow Multi-Agent
2. **PR Review**: Automated analysis and recommendations and human-in-the-loop feedback
3. **Issue Resolution**: Full workflow from analysis to validation
4. **Knowledge Building**: Each resolution enriches the system

## Communication Protocol

### Inter-Agent Messages

```typescript
interface AgentMessage {
  id: string;
  timestamp: string;
  source: {
    agent_id: string;
    type: 'supervisor' | 'worker';
    level: number;
  };
  target: {
    agent_id: string;
    broadcast?: boolean;
  };
  message_type: 'task' | 'result' | 'error' | 'context';
  payload: {
    task_id?: string;
    content: any;
    priority: 1-10;
    requires_response: boolean;
  };
  metadata: {
    correlation_id: string;
    issue_number?: number;
    cht_version?: string;
  };
}
```

## Implementation Phases

### Phase 1: Foundation (Weeks 1-2)
- Set up LangGraph workflow structure
- Implement Master Supervisor routing
- Create basic npm script runner integration
- Establish context file structure

### Phase 2: Core Agents (Weeks 3-4)
- Implement Research agents with CHT docs search
- Build Code Validation Agent with ESLint integration
- Create Test Orchestration Agent
- Develop context persistence layer

### Phase 3: Intelligence Layer (Weeks 5-6)
- Add context recall and pattern matching
- Implement intelligent test selection
- Build recommendation engine
- Create learning feedback loops

### Phase 4: Integration & Testing (Weeks 7-8)
- Full GitHub issue workflow testing
- Performance optimization
- Documentation generation
- Developer experience refinement

## Success Metrics

### Technical Metrics
- Code validation accuracy: >95% alignment with manual review
- Context retrieval relevance: >80% precision for similar issues
- Response time: <60 seconds for standard validations

### Developer Experience Metrics
- False positive rate: <5% for recommendations
- Learning curve: Productive use within 1 day
- Context reuse rate: >60% for similar issues

## Technical Stack

### Core Technologies
- **LangChain JS/LangGraph**: Orchestration framework
- **Claude Models**: Opus for planning, Sonnet 3.5 for implementation
- **Node.js**: Runtime environment (>=20.11.0)
- **TypeScript**: Type safety and developer experience

### CHT Integration Points
- **npm scripts**: All test and lint commands
- **ESLint**: Code quality validation
- **Mocha/Jest/Karma**: Test execution
- **nyc**: Coverage analysis
- **cht-conf**: Configuration management
- **WebdriverIO**: E2E testing

## Risk Mitigation

### Technical Risks
- **Tool API Changes**: Abstract tool interfaces for easy updates
- **Performance Bottlenecks**: Implement caching and parallel execution
- **Context Drift**: Regular validation against actual resolutions
- **False Positives**: Human-in-the-loop for critical decisions

### Mitigation Strategies
- Comprehensive error handling with fallbacks
- Progressive enhancement approach
- Regular context file validation
- Continuous feedback incorporation

## Conclusion

This hierarchical multi-agent system represents a pragmatic approach to enhancing CHT development workflows. By leveraging existing infrastructure, implementing lightweight learning mechanisms, and focusing on immediate value delivery, the system can provide meaningful assistance while remaining maintainable and extensible. The proof of concept prioritizes practical implementation over theoretical completeness, ensuring rapid iteration and real-world validation.