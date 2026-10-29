User clicks "Run crawl"
        │
        ▼
FastAPI starts one background job
        │
        ▼
Crawl (Playwright + axe) → store artifacts
        │
        ▼
Run checks on stored data → save findings
        │
        ▼
Lighthouse on the sample pages → save findings
        │
        ▼
Mark job done