# WorkSpace --- Full-Stack Project Blueprint

> **Project goal:** Build a substantial, production-minded, multi-tenant
> SaaS project management and team collaboration platform. Use it as a
> portfolio project that demonstrates full-stack engineering, system
> design, security, testing, deployment, and the ability to explain
> technical decisions.

**Project type:** Multi-tenant SaaS\
**Category:** Project management + team collaboration\
**Product inspiration:** Trello + Jira + Slack\
**Primary users:** Organizations, managers, and team members\
**Development approach:** Modular, API-driven architecture\
**Initial timeline:** 30-day implementation plan (ambitious; extend it
when needed rather than rushing)

------------------------------------------------------------------------

# 1. Product overview

WorkSpace brings project planning, task management, team communication,
file sharing, and progress tracking into one application.

## Problem it solves

Teams often rely on separate tools for: - Managing projects and
deadlines - Assigning and tracking tasks - Discussing work and sharing
updates - Sharing project documents - Monitoring progress and
productivity

WorkSpace combines these workflows in one platform.

## Main user journey

1.  A user registers and logs in.
2.  The user creates an organization or accepts an invitation.
3.  Organization members create or join projects.
4.  Project members create tasks, assign owners, and set deadlines.
5.  Team members update tasks through a Kanban board.
6.  Members discuss tasks and communicate in project chat.
7.  The application sends notifications about relevant changes.
8.  Managers monitor progress through dashboards and reports.

------------------------------------------------------------------------

# 2. User roles and permissions

The initial application has three roles. Enforce permissions on the
backend; hiding UI controls is not sufficient security.

## Admin --- organization-level control

-   Manage organization settings
-   Invite and remove members
-   Assign organization roles
-   Create and manage projects
-   Access organization-wide analytics
-   Manage organization membership

## Manager --- project-level control

-   Create and manage projects, subject to organization permissions
-   Assign tasks to team members
-   Set deadlines and priorities
-   Monitor project progress
-   Manage project discussions

## Member --- contributor

-   View projects they are authorized to access
-   Update assigned tasks
-   Comment on tasks
-   Upload permitted attachments
-   Communicate with teammates

**Design decision to settle during implementation:** Define precisely
whether Managers can invite members, delete projects, and manage project
membership. Document the chosen permission matrix and test it.

------------------------------------------------------------------------

# 3. Feature specification

## A. Authentication and authorization

-   User registration
-   Login and logout
-   Password hashing with bcrypt
-   JWT access tokens
-   Refresh tokens and rotation
-   Refresh-token revocation
-   Protected API routes
-   Secure cookie handling where appropriate
-   Forgot-password and password-reset flow (later milestone)
-   Role-based access control (RBAC)
-   Session management

**Concepts:** Authentication vs authorization, JWT, cookies, middleware,
password hashing, token expiry and rotation.

## B. Organization management

-   Create and edit an organization
-   Organization settings
-   Invite members through email
-   Accept or reject invitations
-   Assign and update roles
-   Remove members
-   Switch between organizations
-   Organization-level data isolation

**Multi-tenancy:** Multiple organizations use the same application and
infrastructure, but one organization must never access another
organization's private data. Every relevant query and mutation must be
scoped to an authorized organization.

## C. Project management

-   Create, view, edit, and delete projects
-   Assign project managers
-   Add and remove project members
-   Set project descriptions and dates
-   Track project status
-   Search and filter projects
-   Sort and paginate project lists
-   Enforce project-level permissions

## D. Task management

-   Create, view, edit, and delete tasks
-   Assign tasks to users
-   Set due dates
-   Set priorities: Low, Medium, High, Critical
-   Set statuses: To Do, In Progress, In Review, Completed
-   Add descriptions
-   Add comments
-   Track task history
-   Filter by status, priority, assignee, and deadline

## E. Kanban board

-   Four initial columns: To Do, In Progress, In Review, Completed
-   Drag and drop task cards
-   Move tasks between columns
-   Persist status and ordering in the database
-   Optimistic UI updates with rollback on failure
-   Show priority, assignee, and due date on cards
-   Enforce permissions when moving tasks

## F. Task collaboration

-   Detailed task view
-   Comments and replies (replies can be a later enhancement)
-   File attachments
-   Activity history
-   Assignment and deadline change history
-   Member mentions (later enhancement)
-   Timestamps

## G. Real-time notifications

Notify users when: - They are assigned a task - A task deadline or
status changes - Someone comments on a task they follow or own - They
receive an organization invitation - A project they belong to is
updated - Someone mentions them

Implementation ideas: - Socket.IO - Authenticated user-specific rooms -
Persist notifications in the database - Read/unread status - Unread
notification counter - Handle reconnects and missed events by fetching
persisted notifications

## H. Team chat

-   Project-specific chat rooms
-   Real-time messaging
-   Persistent message history
-   Sender and timestamp information
-   Room-level access control
-   Message pagination/history loading
-   Online presence as a later enhancement

## I. Search and filtering

-   Search projects, tasks, and team members
-   Filter by status, priority, assignee, and date
-   Sort by creation date and deadline
-   Server-side pagination
-   Validate query parameters

## J. Analytics dashboard

Initial metrics: - Total, active, and completed projects - Total,
pending, overdue, and completed tasks - Project completion percentage -
Task status distribution - Weekly task completion - Tasks completed per
member

Use charts only where they help users understand progress. Ensure
analytics queries are scoped to the user's authorized organization and
projects.

## K. File management

-   Upload project documents and task attachments
-   Download and delete authorized files
-   Validate file type and size
-   Store files in AWS S3
-   Use pre-signed URLs where appropriate
-   Authorize access before issuing download/upload URLs
-   Avoid exposing permanent public file URLs for private content

## L. Background jobs and email

-   Send organization invitations
-   Send task assignment emails
-   Send deadline reminders
-   Process notifications asynchronously where useful
-   Retry failed jobs
-   Track failed jobs and provide operational visibility

Suggested tools: Redis + BullMQ + an email service.

## M. Export and reporting

-   Export project task lists as CSV
-   Generate PDF project reports
-   Export analytics summaries
-   Filter reports by project and date range

------------------------------------------------------------------------

# 4. Technology stack

This is the proposed stack. Introduce technologies only when the
corresponding feature needs them.

## Frontend

-   **Next.js + React + TypeScript:** UI, routing, reusable components,
    type safety
-   **Tailwind CSS:** styling
-   **shadcn/ui:** accessible reusable UI components
-   **TanStack Query:** server state, caching, and API synchronization
-   **Drag-and-drop library:** Kanban interactions (choose and document
    one)

## Backend

-   **Node.js + Express.js + TypeScript:** REST APIs and business logic
-   **Zod:** request and input validation
-   **Socket.IO:** real-time communication
-   **Prisma:** database access and schema migrations

## Database and infrastructure

-   **PostgreSQL:** relational database
-   **Redis:** caching and job infrastructure
-   **BullMQ:** background jobs
-   **AWS S3:** object/file storage

## Testing and delivery

-   **Jest:** unit tests
-   **Supertest:** API tests
-   **Playwright:** end-to-end tests
-   **Docker + Docker Compose:** reproducible local environment
-   **Git + GitHub:** version control
-   **GitHub Actions:** CI/CD
-   **Render or AWS:** deployment target (choose based on cost and
    complexity)

------------------------------------------------------------------------

# 5. High-level architecture

## Request flow

1.  User interacts with the Next.js frontend.
2.  Frontend sends an HTTP request to the Express backend.
3.  Backend middleware validates authentication.
4.  Authorization middleware checks role, organization membership, and
    resource access.
5.  Controller validates request data and calls the service layer.
6.  Service executes business logic and uses Prisma to access
    PostgreSQL.
7.  Backend returns a response to the frontend.
8.  Socket.IO emits relevant real-time events.
9.  Redis supports caching and background-job infrastructure.
10. BullMQ workers process asynchronous tasks.
11. AWS S3 stores uploaded files.

## Suggested backend layers

-   **Routes:** Map HTTP methods and paths to controllers.
-   **Middleware:** Authentication, authorization, validation, rate
    limiting, and error handling.
-   **Controllers:** Handle HTTP request/response concerns.
-   **Services:** Contain business rules and orchestrate operations.
-   **Database layer:** Prisma queries and transactions.
-   **Workers:** Process background jobs.
-   **Socket layer:** Authenticated real-time connections and event
    handling.

Keep the architecture understandable. Avoid creating abstractions that
do not solve a real problem.

------------------------------------------------------------------------

# 6. Database design

Initial relational tables/models:

  Table / Model   Purpose
  --------------- ---------------------------------------------------------
  User            User accounts and profile information
  Organization    Company or team workspace
  Membership      User-to-organization relationship and organization role
  Invitation      Pending invitations, expiry, and acceptance state
  Project         Projects owned by an organization
  ProjectMember   Project-level membership and permissions if needed
  Task            Tasks within projects
  Comment         Task discussions
  Attachment      File metadata and storage key
  ActivityLog     Record of important actions
  Notification    User notifications and read status
  Message         Project chat messages
  RefreshToken    Session/token management

## Important relationships

-   One organization has many memberships.
-   One user can belong to multiple organizations.
-   One organization has many projects.
-   A project can have many project members.
-   One project has many tasks.
-   One task can have many comments.
-   One task can have multiple attachments.
-   One user can be assigned many tasks.
-   One project can have many messages.
-   One user can receive many notifications.

## Database design questions to resolve

-   Should task assignment allow one assignee or multiple assignees?
-   Should project membership be explicit, or should all organization
    members see all projects?
-   What happens to tasks when a member is removed?
-   Should deleted projects/tasks be soft-deleted?
-   Which actions require a database transaction?
-   Which columns need indexes for common filters and joins?

Document these decisions before implementation.

------------------------------------------------------------------------

# 7. Initial REST API plan

  Method   Endpoint                        Purpose
  -------- ------------------------------- ---------------------------
  POST     `/api/auth/register`            Register
  POST     `/api/auth/login`               Login
  POST     `/api/auth/refresh`             Refresh access token
  POST     `/api/auth/logout`              Logout
  GET      `/api/organizations`            List user's organizations
  POST     `/api/organizations`            Create organization
  POST     `/api/invitations`              Invite members
  GET      `/api/projects`                 List authorized projects
  POST     `/api/projects`                 Create project
  GET      `/api/projects/:id`             Get project details
  PATCH    `/api/projects/:id`             Update project
  DELETE   `/api/projects/:id`             Delete project
  GET      `/api/tasks`                    List authorized tasks
  POST     `/api/tasks`                    Create task
  GET      `/api/tasks/:id`                Get task details
  PATCH    `/api/tasks/:id`                Update task
  DELETE   `/api/tasks/:id`                Delete task
  POST     `/api/tasks/:id/comments`       Add comment
  GET      `/api/notifications`            List notifications
  PATCH    `/api/notifications/:id/read`   Mark notification as read
  GET      `/api/analytics`                Dashboard analytics

These are starting examples, not a frozen contract. Add endpoints for
invitations, memberships, attachments, messages, and exports as their
modules are designed.

------------------------------------------------------------------------

# 8. Security requirements

Security must be implemented as part of each feature, not added at the
end.

-   Hash passwords with bcrypt.
-   Validate all incoming data with Zod or equivalent validation.
-   Use short-lived access tokens and carefully managed refresh tokens.
-   Store refresh tokens securely; prefer HTTP-only, Secure, SameSite
    cookies where suitable.
-   Apply authentication middleware to protected endpoints.
-   Apply role-based and resource-level authorization.
-   Enforce organization/tenant scoping in every relevant database
    query.
-   Prevent insecure direct object references (IDOR).
-   Rate-limit sensitive endpoints such as login and invitation
    creation.
-   Configure CORS narrowly.
-   Keep secrets in environment variables.
-   Validate uploaded file type, size, and access rights.
-   Use database constraints and indexes.
-   Avoid returning stack traces or sensitive internal errors to
    clients.
-   Log security-relevant events without logging passwords or tokens.

## Essential security test cases

-   User from Organization A cannot read Organization B's project.
-   Member cannot perform Admin-only actions.
-   User cannot update a task they are not authorized to access.
-   Expired invitation cannot be accepted.
-   Revoked refresh token cannot create a new session.
-   Unauthorized user cannot obtain a file download URL.
-   A user cannot access another user's private notification data.

------------------------------------------------------------------------

# 9. Testing strategy

  -----------------------------------------------------------------------
  Test type               What to test            Tool
  ----------------------- ----------------------- -----------------------
  Unit tests              Individual functions    Jest
                          and business rules      

  API tests               HTTP endpoints and      Supertest
                          status codes            

  Integration tests       Database/service        Jest + test database
                          interactions            

  E2E tests               Complete user workflows Playwright

  Security tests          Authorization and       Automated and manual
                          tenant isolation        tests
  -----------------------------------------------------------------------

## Minimum end-to-end workflows

1.  Register → login → create organization.
2.  Invite member → accept invitation → verify membership.
3.  Create project → add member → create task.
4.  Move task across Kanban columns → refresh → verify persistence.
5.  Add comment → verify authorized users can see it.
6.  Trigger notification → mark it read.
7.  Attempt cross-organization access → verify it is denied.

A feature is not complete merely because the happy path works. Test
invalid input, missing permissions, missing resources, and service
failures.

------------------------------------------------------------------------

# 10. Deployment and DevOps

-   Create frontend and backend Dockerfiles.
-   Use Docker Compose for local services such as PostgreSQL and Redis.
-   Keep development, test, and production environment variables
    separate.
-   Run database migrations safely during deployment.
-   Add GitHub Actions for linting, type checking, and tests.
-   Deploy frontend and backend.
-   Configure production database and storage.
-   Configure CORS, cookies, and HTTPS correctly.
-   Add health-check endpoints.
-   Add structured logging and basic error monitoring.
-   Document backup and recovery expectations.

------------------------------------------------------------------------

# 11. 30-day implementation roadmap

> Suggested daily effort: approximately 3 hours. This is an ambitious
> plan. If a concept or feature needs more time, extend the schedule
> instead of marking incomplete work as done.

## Week 1 --- Foundation

-   [ ] **Day 1:** Understand SaaS, roles, multi-tenancy, entity
    relationships, and system architecture. Draw the first architecture
    diagram.
-   [ ] **Day 2:** Set up Next.js, Express, TypeScript, Tailwind,
    environment variables, and a test API connection.
-   [ ] **Day 3:** Learn PostgreSQL and Prisma. Connect the backend to
    the database.
-   [ ] **Day 4:** Design User, Organization, Membership, and Invitation
    schemas. Learn constraints, relationships, migrations, and
    normalization.
-   [ ] **Day 5:** Set up routes, controllers, services, middleware, Zod
    validation, centralized error handling, and API testing.

## Week 2 --- Authentication

-   [ ] **Day 6:** Implement registration, password hashing, validation,
    and duplicate-email handling.
-   [ ] **Day 7:** Implement login, JWT access tokens, authentication
    middleware, and protected APIs.
-   [ ] **Day 8:** Implement refresh tokens, rotation, logout,
    revocation, and secure cookie handling.
-   [ ] **Day 9:** Build frontend authentication pages, connect APIs,
    manage auth state, and protect routes.
-   [ ] **Day 10:** Implement Admin/Manager/Member RBAC and test
    permission boundaries.

## Week 3 --- Core product

-   [ ] **Day 11:** Build organization management and enforce tenant
    isolation.
-   [ ] **Day 12:** Implement invitations, acceptance, expiry, and role
    assignment.
-   [ ] **Day 13:** Build project CRUD, membership, permissions,
    filtering, and pagination.
-   [ ] **Day 14:** Build task CRUD, assignment, deadlines, priority,
    status, sorting, and filtering.
-   [ ] **Day 15:** Build the main dashboard UI.
-   [ ] **Day 16:** Implement Kanban drag-and-drop, backend
    synchronization, optimistic updates, and rollback.
-   [ ] **Day 17:** Add task details, comments, attachments, and
    activity history.

## Week 4 --- Advanced features

-   [ ] **Day 18:** Learn Socket.IO, rooms, authentication, and
    connection lifecycle.
-   [ ] **Day 19:** Implement real-time notifications, persistence, and
    unread counts.
-   [ ] **Day 20:** Build project chat and message persistence.
-   [ ] **Day 21:** Add Redis caching and cache invalidation.
-   [ ] **Day 22:** Add BullMQ background jobs, retries, and failure
    handling.
-   [ ] **Day 23:** Integrate AWS S3 and secure file uploads/downloads.
-   [ ] **Day 24:** Build analytics APIs and dashboard charts.

## Week 5 --- Quality and launch

-   [ ] **Day 25:** Write backend unit and API tests for authentication,
    projects, tasks, RBAC, and errors.
-   [ ] **Day 26:** Write frontend and Playwright end-to-end tests.
-   [ ] **Day 27:** Perform security and performance checks: rate
    limiting, validation, IDOR, tenant isolation, and database indexes.
-   [ ] **Day 28:** Create Dockerfiles and Docker Compose configuration
    for the application and dependencies.
-   [ ] **Day 29:** Deploy the application, configure production
    environment variables, and set up GitHub Actions CI/CD.
-   [ ] **Day 30:** Complete README, architecture documentation, API
    documentation, database relationship diagram, screenshots, demo, and
    interview explanations.

------------------------------------------------------------------------

# 12. Daily work routine

Suggested 3-hour structure:

  -----------------------------------------------------------------------
  Time                                Activity
  ----------------------------------- -----------------------------------
  45 minutes                          Learn the concepts required for
                                      today's feature

  90 minutes                          Implement with AI assistance, one
                                      small component at a time

  45 minutes                          Run, test, debug, review, and
                                      explain the implementation
  -----------------------------------------------------------------------

## AI-assisted development workflow

1.  **Learn:** Understand the concept before asking for code.
2.  **Design:** Ask AI to explain the architecture, data flow, and files
    involved.
3.  **Generate:** Implement one small feature at a time. Avoid
    generating the entire application at once.
4.  **Understand:** Ask for explanations of functions, components, API
    calls, database queries, and error paths.
5.  **Verify:** Run the application and test the feature yourself.
6.  **Reproduce:** Try to explain or rebuild the feature without AI.

## Questions to ask about every important piece of code

-   What problem does this code solve?
-   Why was this approach chosen?
-   What happens if the API fails?
-   What happens if two users update the same task?
-   Where is the data stored?
-   How are permissions checked?
-   What are the edge cases?
-   What would break if this component were removed?
-   How would I test this?
-   How would I explain this in an interview?

------------------------------------------------------------------------

# 13. What this project demonstrates on a resume

  -----------------------------------------------------------------------
  Skill area                          Evidence from the project
  ----------------------------------- -----------------------------------
  Frontend                            Next.js, React, TypeScript,
                                      responsive UI

  Backend                             Express, REST APIs, middleware,
                                      service architecture

  Database                            PostgreSQL, Prisma, relational
                                      schema design

  Authentication                      JWT, refresh tokens, RBAC

  System design                       Multi-tenancy, modular
                                      architecture, data isolation

  Real-time systems                   Socket.IO, notifications, chat

  Caching                             Redis

  Background processing               BullMQ

  Cloud                               AWS S3

  Testing                             Jest, Supertest, Playwright

  DevOps                              Docker, CI/CD, deployment

  Security                            Authorization, tenant isolation,
                                      validation, IDOR prevention
  -----------------------------------------------------------------------

## Draft resume entry --- use only after implementation

**WorkSpace --- Multi-Tenant Project Management & Collaboration
Platform** \| Full-Stack Developer

-   Designed and developed a multi-tenant SaaS platform for organization
    management, project tracking, task assignment, and team
    collaboration.
-   Implemented secure authentication, JWT-based sessions, role-based
    access control, and organization-level data isolation.
-   Built interactive Kanban boards, task workflows, real-time
    notifications, and team messaging.
-   Integrated PostgreSQL, Redis, background job processing, and AWS S3
    for persistent data, caching, asynchronous tasks, and file storage.
-   Added automated testing, Docker-based development, and CI/CD
    deployment workflows.

**Proposed tech stack:** Next.js, React, TypeScript, Node.js,
Express.js, PostgreSQL, Prisma, Redis, Socket.IO, BullMQ, AWS S3,
Docker, Jest, Playwright.

**Important:** Do not put technologies or features on the resume until
they are genuinely implemented and you can explain them.

------------------------------------------------------------------------

# 14. Future enhancements --- not part of the initial core

Build these only after the core product is stable:

-   AI project assistant
-   AI-generated task summaries
-   GitHub integration
-   Calendar integration
-   Video meetings
-   Subscription and billing
-   Enterprise audit and compliance features
-   Mobile application

The priority is a dependable core application, not the maximum number of
features.

------------------------------------------------------------------------

# 15. Definition of done

A feature is complete only when:

-   [ ] The feature works in the application.
-   [ ] Its API and data flow are understood.
-   [ ] Input validation and error handling exist.
-   [ ] Authorization rules are enforced on the backend.
-   [ ] Relevant success and failure cases are tested.
-   [ ] The code is committed to Git.
-   [ ] The README or technical notes are updated.
-   [ ] You can explain the implementation without relying on AI.

------------------------------------------------------------------------

# 16. Day 1 --- Start here

Do not generate the application code yet. First understand these five
topics:

1.  What is a SaaS application?
2.  What is multi-tenancy?
3.  How are User, Organization, Project, and Task related?
4.  What is the difference between frontend, backend, and database?
5.  How does a request travel from the browser to the database and back?

## Day 1 deliverables

-   [ ] Write a short explanation of WorkSpace in your own words.
-   [ ] Draw the relationship: Organization → Projects → Tasks.
-   [ ] Draw how Users connect to Organizations through Membership.
-   [ ] Draw a simple frontend → backend → database request flow.
-   [ ] List the permissions of Admin, Manager, and Member.
-   [ ] Create a GitHub repository and initial README after
    understanding the design.

**Guiding principle:** The goal is not merely to put a large project on
your resume. The goal is to understand the system well enough to explain
its architecture, trace its data flow, test its failure cases, and
defend its technical decisions.
