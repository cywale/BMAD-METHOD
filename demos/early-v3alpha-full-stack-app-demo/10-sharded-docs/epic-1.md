# Epic 1: Project Initialization, Setup, and HN Content Acquisition

> This document is a granulated shard from the main "BETA-V3/v3-demos/full-stack-app-demo/8-prd-po-updated.md" focusing on "Epic 1: Project Initialization, Setup, and HN Content Acquisition".

- Goal: Establish the foundational project structure, including the Next.js application, Supabase integration, deployment pipeline, API/CLI triggers, core workflow orchestration, and implement functionality to retrieve, process, and store Hacker News posts/comments via a `ContentAcquisitionFacade`, providing data for newsletter generation. Implement the database event mechanism to trigger subsequent processing. Define core configuration tables, seed data, and set up testing frameworks.

- **Story 1.1:** As a developer, I want to set up the Next.js project with Supabase integration, so that I have a functional foundation for building the application.
  - Acceptance Criteria:
    - The Next.js project is initialized using the Vercel/Supabase template.
    - Supabase is successfully integrated with the Next.js project.
    - The project codebase is initialized in a Git repository.
    - A basic project `README.md` is created in the root of the repository, including a project overview, links to main documentation (PRD, architecture), and essential developer setup/run commands.
- **Story 1.2:** As a developer, I want to configure the deployment pipeline to Vercel with separate development and production environments, so that I can easily deploy and update the application.
  - Acceptance Criteria:
    - The project is successfully linked to a Vercel project with separate environments.
    - Automated deployments are configured for the main branch to the production environment.
    - Environment variables are set up for local development and Vercel deployments.
- **Story 1.3:** As a developer, I want to implement the API and CLI trigger mechanisms, so that I can manually trigger the workflow during development and testing.
  - Acceptance Criteria:
    - A secure API endpoint is created.
    - The API endpoint requires authentication (API key).
    - The API endpoint (`/api/system/trigger-workflow`) creates an entry in the `workflow_runs` table and returns the `jobId`.
    - The API endpoint returns an appropriate response to indicate success or failure.
    - The API endpoint is secured via an API key.
    - A CLI command is created.
    - The CLI command invokes the `/api/system/trigger-workflow` endpoint or directly interacts with `WorkflowTrackerService` to start a new workflow run.
    - The CLI command provides informative output to the console.
    - All API requests and CLI command executions are logged, including timestamps and any relevant data.
    - All interactions with the API or CLI that initiate a workflow must record the `workflow_run_id` in logs.
    - The API and CLI interfaces adhere to mobile responsiveness and Tailwind/theming principles.
- **Story 1.4:** As a system, I want to retrieve the top 30 Hacker News posts and associated comments daily using a configurable `ContentAcquisitionFacade`, so that the data is available for summarization and newsletter generation.
  - Acceptance Criteria:
    - A `ContentAcquisitionFacade` is implemented in `supabase/functions/_shared/` to abstract interaction with the news data source (initially HN Algolia API).
    - The facade handles API authentication (if any), request formation, and response parsing for the specific news source.
    - The facade implements basic retry logic for transient errors.
    - Unit tests for the `ContentAcquisitionFacade` (mocking actual HTTP calls to the HN Algolia API) achieve >80% coverage.
    - The system retrieves the top 30 Hacker News posts daily via the `ContentAcquisitionFacade`.
    - The system retrieves associated comments for the top 30 posts via the `ContentAcquisitionFacade`.
    - Retrieved data (posts and comments) is stored in Supabase database, linked to the current `workflow_run_id`.
    - This functionality can be triggered via the API and CLI.
    - The system logs the start and completion of the retrieval process, including any errors.
    - Upon completion, the service updates the `workflow_runs` table with status and details (e.g., number of posts fetched) via `WorkflowTrackerService`.
    - Supabase migrations for `hn_posts` and `hn_comments` tables (as defined in `architecture.txt`) are created and applied before data operations.
- **Story 1.5: Define and Implement `workflow_runs` Table and `WorkflowTrackerService`**
  - Goal: Implement the core workflow orchestration mechanism (tracking part).
  - Acceptance Criteria:
    - Supabase migration created for the `workflow_runs` table as defined in the architecture document.
    - `WorkflowTrackerService` implemented in `supabase/functions/_shared/` with methods for initiating, updating step details, incrementing counters, failing, and completing workflow runs.
    - Service includes robust error handling and logging via Pino.
    - Unit tests for `WorkflowTrackerService` achieve >80% coverage.
- **Story 1.6: Implement `CheckWorkflowCompletionService` (Supabase Cron Function)**
  - Goal: Implement the core workflow orchestration mechanism (progression part).
  - Acceptance Criteria:
    - Supabase Function `check-workflow-completion-service` created.
    - Function queries `workflow_runs` and related tables to determine if a workflow run is ready to progress to the next major stage.
    - Function correctly updates `workflow_runs.status` and invokes the next appropriate service function.
    - Logic for handling podcast link availability is implemented here or in conjunction with `NewsletterGenerationService`.
    - The function is configurable to be run periodically.
    - Comprehensive logging implemented using Pino.
    - Unit tests achieve >80% coverage.
- **Story 1.7: Implement Workflow Status API Endpoint (`/api/system/workflow-status/{jobId}`)**
  - Goal: Allow developers/admins to check the status of a workflow run.
  - Acceptance Criteria:
    - Next.js API Route Handler created at `/api/system/workflow-status/{jobId}`.
    - Endpoint secured with API Key authentication.
    - Retrieves and returns status details from the `workflow_runs` table.
    - Handles cases where `jobId` is not found (404).
    - Unit and integration tests for the API endpoint.
- **Story 1.8: Create and document `docs/environment-vars.md` and set up `.env.example`**
  - Goal: Ensure environment variables are properly documented and managed.
  - Acceptance Criteria:
    - A `docs/environment-vars.md` file is created.
    - An `.env.example` file is created.
    - Sensitive information in examples is masked.
    - For each third-party service requiring credentials, `docs/environment-vars.md` includes:
      - A brief note or link guiding the user on where to typically sign up for the service and obtain the necessary API key or credential.
      - A recommendation for the user to check the service's current free/low-tier API rate limits against expected MVP usage.
      - A note that usage beyond free tier limits for commercial services (like Play.ht, remote LLMs, or email providers) may incur costs, and the user should review the provider's pricing.
- **Story 1.9 (New): Implement Database Event/Webhook: `hn_posts` Insert to Article Scraping Service**
  - Goal: To ensure that the successful insertion of a new Hacker News post into the `hn_posts` table automatically triggers the `ArticleScrapingService`.
  - Acceptance Criteria:
    - A Supabase database trigger or webhook mechanism (e.g., using `pg_net` or native triggers calling a function) is implemented on the `hn_posts` table for INSERT operations.
    - The trigger successfully invokes the `ArticleScrapingService` (Supabase Function).
    - The invocation passes necessary parameters like `hn_post_id` and `workflow_run_id` to the `ArticleScrapingService`.
    - The mechanism is robust and includes error handling/logging for the trigger/webhook itself.
    - Unit/integration tests are created to verify the trigger fires correctly and the service is invoked with correct parameters.
- **Story 1.10 (New): Define and Implement Core Configuration Tables**
  - Goal: To establish the database tables necessary for storing core application configurations like summarization prompts, newsletter templates, and subscriber lists.
  - Acceptance Criteria:
    - A Supabase migration is created and applied to define the `summarization_prompts` table schema as specified in `architecture.txt`.
    - A Supabase migration is created and applied to define the `newsletter_templates` table schema as specified in `architecture.txt`.
    - A Supabase migration is created and applied to define the `subscribers` table schema as specified in `architecture.txt`.
    - These tables are ready for data population (e.g., via seeding or manual entry for MVP).
- **Story 1.11 (New): Create Seed Data for Initial Configuration**
  - Goal: To populate the database with initial configuration data (prompts, templates, test subscribers) necessary for development and testing of MVP features.
  - Acceptance Criteria:
    - A `supabase/seed.sql` file (or an equivalent, documented seeding mechanism) is created.
    - The seed mechanism populates the `summarization_prompts` table with at least one default article prompt and one default comment prompt.
    - The seed mechanism populates the `newsletter_templates` table with at least one default newsletter template (HTML format for MVP).
    - The seed mechanism populates the `subscribers` table with a small list of 1-3 test email addresses for MVP delivery testing.
    - Instructions on how to apply the seed data to a local or development Supabase instance are documented (e.g., in the project `README.md`).
- **Story 1.12 (New): Set up and Configure Project Testing Frameworks**
  - Goal: To ensure that the primary testing frameworks (Jest, React Testing Library, Playwright) are installed and configured early in the project lifecycle, enabling test-driven development practices and adherence to the testing strategy.
  - Acceptance Criteria:
    - Jest and React Testing Library (RTL) are installed as project dependencies.
    - Jest and RTL are configured for unit and integration testing of Next.js components and JavaScript/TypeScript code (e.g., `jest.config.js` is set up, necessary Babel/TS transformations are in place).
    - A sample unit test (e.g., for a simple component or utility function) is created and runs successfully using the Jest/RTL setup.
    - Playwright is installed as a project dependency.
    - Playwright is configured for end-to-end testing (e.g., `playwright.config.ts` is set up, browser configurations are defined).
    - A sample E2E test (e.g., navigating to the application's homepage on the local development server) is created and runs successfully using Playwright.
    - Scripts to execute tests (e.g., unit tests, E2E tests) are added to `package.json`.
