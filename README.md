# Search and Indexing Decisions for StatWrap

## Index In memory or store locally
The problem statement mentions that indexing must be done locally due to sensitive data. Should the index be kept entirely in-memory or can it be stored persistently (e.g., in a local file or database)? Understanding this will help optimize search performance based on the expected project size.

## Central or Isolated Index
I've noticed StatWrap currently maintains project isolation. For the search feature, would you prefer an isolated search approach that respects this isolation or a centralized index that might offer better performance? 

### Current Situation
StatWrap loads and works with one project at a time (in `app/main.dev.js`). Each project's data is isolated in its own directory structure, so projects aren't aware of each other's content. 

### Why This Matters
- **Federated approach**: Search each project individually, maintaining the current isolation model.
- **Centralized approach**: Build one master index, faster but architectural shift.
- **Hybrid possibility**: Central metadata index with separate content indices.

### Implementation Impact
This fundamentally affects where your indices are stored, how they're updated, and the search query flow. A centralized index is faster but breaks the current architecture pattern.

## Search Functionality in Worker or Main Process
Should the search functionality be implemented primarily in the main process or worker process? I'm considering the trade-offs between UI responsiveness and search performance.

### Current Situation
StatWrap uses a worker window for CPU-intensive tasks (as shown in `app/main.dev.js`). The UI runs in the main renderer process, and assets are scanned in background processes.

### Why This Matters
- **Main process**: Simpler implementation but could freeze UI during searches.
- **Worker process**: More complex but maintains UI responsiveness. Builds on StatWrap's existing pattern of offloading intensive work.

### Implementation Impact
This affects your IPC message design, how search queries are processed, and whether you need synchronization between processes.

## Incremental or Batch Indexing
Would you prefer incremental indexing that updates as files change, or batch indexing when projects are closed/opened to minimize performance impact during active work?

### Current Situation
StatWrap has a `LogWatcherService` that monitors project changes. Full project scans happen when projects are loaded, but there is no evidence of background processing during editing.

### Why This Matters
- **Incremental**: Always up-to-date but constant background work.
- **Batch**: Predictable performance impact, concentrated at specific times.
- **Hybrid**: Small updates in real-time, full reindex during idle times.

### Implementation Impact
This affects when your indexing code runs, how you detect changes, and how you handle partial updates to the index. It has significant performance implications.

## Search Index Persistence Strategy
What's the preferred approach for persisting search indices? I've considered SQLite for structured queries, JSON for simplicity, or a specialized binary format for performance. How should we balance size, load time, and query performance?

### Current Situation
- StatWrap stores project metadata in JSON files.
- No evidence of a database system for handling large datasets.
- Project data is primarily file-based.

### Why This Matters
- Different persistence strategies have vastly different performance profiles.
- Some approaches are more resilient to corruption or app crashes.
- Storage format affects how efficiently you can update portions of the index.

### Implementation Impact
This impacts your storage backend, file format choices, and error recovery mechanisms.

## What to Do After Search
When a user finds a result in a project that isn't currently open, what's the preferred user experience? Should we provide a live preview by pre-loading project metadata or immediately navigate to the file? What's the preferred workflow for researchers?

### Current Situation
- StatWrap loads one project at a time.
- Switching projects appears to be a heavyweight operation.
- No visible mechanism for multi-project awareness.

### Why This Matters
- Affects user experience when navigating search results.
- Raises questions about resource usage and memory management.
- Might require new UI patterns not currently in the application.

### Implementation Impact
This influences how search results link to actual content and what metadata must be maintained outside of project loading.

## Search UI Integration
For the search UI, would you prefer a global search bar with dropdown results, or a dedicated search page with more advanced filtering options?

### Current Situation
- StatWrap has a navigation header where a search bar could fit.
- No existing search UI patterns to follow.
- Application uses Material UI components based on code samples.

### Why This Matters
- **Global search**: Quick access, limited filtering, inline results.
- **Dedicated page**: More powerful, better for complex filtering, takes more clicks.
- **Hybrid approach**: Quick search with "advanced search" option.

### Implementation Impact
This affects your component hierarchy, navigation flow, and overall UX design.

## Search Security Model
Given that research projects might contain sensitive information, how should we approach security for the search index? Should indices be encrypted, and should we implement any content exclusion mechanisms?

### Current Situation
- In the project description, there is mention of "the potentially sensitive nature of data," but there is no obvious encryption mechanism visible in the codebase.
- Projects are stored locally, providing some isolation.

### Why This Matters
- Search indices contain extracted content that might include sensitive data.
- Different approaches to security have performance implications.
- May influence which fields are indexed and how deeply.

### Implementation Impact
This affects your encryption strategy, potential exclusion patterns, and how search results are displayed.

## Filters for Search
Beyond Active/Past classification, what other metadata would be most valuable for filtering search results? Language, last modified date, or something else?

### Current Situation
- StatWrap tracks various metadata about projects and files.
- Assets are already categorized by language/type.
- Projects currently don't have an Active/Past status (we'll be adding this).

### Why This Matters
- Good filters will improve search usability.
- Different users might have different search strategies.
- Too many filters can overwhelm the interface, too few limits usefulness.

### Implementation Impact
This affects your database schema, what fields you index, and the UI complexity of your search interface.

## Search Result Ranking Algorithm
What factors should influence search result ranking? Beyond basic text matching, should we consider file types, recency, project status, or user-defined importance tags when scoring results?

### Current Situation
- No existing search functionality to reference, but projects have metadata like status (Active/Past) that we'll be adding.
- Files have different types and importance in the workflow.

### Why This Matters
- Search is only as good as its relevance ranking.
- Different ranking algorithms prioritize different aspects.
- Custom ranking requires more implementation effort but provides better results.
