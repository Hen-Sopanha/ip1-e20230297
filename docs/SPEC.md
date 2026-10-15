# TicketDesk: Specification

## 1. Purpose
TicketDesk is a web app for reporting and resolving support problems.
- Users open tickets to describe and report a problem.
- Agents pick up those tickets and work on them through a comment thread and a status flow. 
- Admins manage the agents and categories and track the overall workload.

The goal is to replace scattered requests (messages, calls, word of mouth)
with one system where every problem is recorded, has a clear owner, and has a
visible status from Open to Closed.

**Intended users:**
- People who have a problem and need to report it (users)
- Support staff who solve those problems (agents)
- A manager who oversees the support team (admin)

**Example setting:** a university IT office, where students report issues
such as Wi-Fi login failures and IT staff resolve them.

## 2. Roles
| Role  | Description |
|-------|-------------|
| User  | Opens tickets and sees only their own |
| Agent | Handles tickets by assigns, replies, changes status |
| Admin | Manages agents and categories, sees the summary |

## 3. Scope
**In scope:** tickets, comments, status flow, assignment, categories, admin summary
**Out of scope:** email notifications, real-time chat, SLA timers, automatic closing of Resolved tickets

## 4. Data entities
- User: id, name, email, passwordHash, role (User, Agent, Admin), isActive
- Ticket: id, title, description, priority (Low, Medium, High), status (Open, In Progress, Resolved, Closed), createdBy, assignedTo, resolvedAt, categoryId
- Comment: id, ticketId, authorId, body, createdAt
- Category: id, name
- Relations: User 1-N Tickets, Ticket 1-N Comments, Category 1-N Tickets

## 5. Business rules
- Status transitions:
  | From | To | Who |
  |------|----|-----|
  | Open | In progress | Agent (assigns to self) |
  | In progress | Resolved | Assigned agent |
  | Resolved | Closed | Assigned agent |
  | Resolved | In progress | Assigned agent |
  | Closed | none | |
- When a ticket becomes Resolved, `resolvedAt` is set and the user has 2 days to reply
- The agent closes the ticket when the user confirms the fix, or after 2 days with no reply
- A closed ticket cannot be reopened and cannot receive comments
- When a ticket is Resolved and the user replies that the problem is not fixed, the assigned agent moves it back to In progress (`resolvedAt` is cleared)
- A user sees only their own tickets. An agent sees unassigned tickets and tickets assigned to them. An admin sees all tickets. Anyone else gets 404
- Comments cannot be edited or deleted
## 6. User stories

### US-1 (User): Create a ticket
As a user, I can create a ticket with a title, description and priority, so the IT team knows my problem.
- Given I am logged in, when I submit valid data, then a ticket is created with status Open
- Given the title is empty, when I submit, then I see "Title is required" and nothing is saved

### US-2 (User): View my tickets
As a user, I can see only my own tickets, so my requests stay private.
- Given I am logged in, when I open the ticket list, then I see only tickets I created
- Given a ticket belongs to another user, when I open it by id, then I get 404

### US-3 (User): Comment on my ticket
As a user, I can comment on my ticket, so I can reply to the agent.
- Given my ticket is not Closed, when I submit a comment, then it appears in the thread
- Given my ticket is Resolved, when I comment "Fixed" or "Still not working", then the agent can see my answer
- Given my ticket is Closed, when I try to comment, then I see "Ticket is closed"

### US-4 (Agent): Assign a ticket to myself
As an agent, I can assign an open ticket to myself, so everyone knows who is handling it.
- Given a ticket is Open and unassigned, when I assign it to myself, then its status becomes In progress
- Given a ticket is already assigned, when I try to assign it, then I see "Already assigned"
- Given I am the assigned agent, when I submit a comment, then it appears in the thread

### US-5 (Agent): Change ticket status
As an agent, I can change the status of my assigned tickets, so users see progress.
- Given I am the assigned agent and the ticket is Resolved, when I set it to Closed, then the status is Closed
- Given a ticket is Open, when I try to set it directly to Resolved, then I see "Invalid status change"
- Given a ticket is assigned to another agent, when I try to change its status, then I get 404
- Given I am a user, when I try to change a ticket's status, then I get 403
- Given a ticket is Closed, when I try to reopen it, then I see "Closed tickets cannot be reopened"
- Given a ticket is Resolved and the user says it is still not fixed, when I set the status to In progress, then the status is updated and `resolvedAt` is cleared

### US-6 (Agent): Filter my work list
As an agent, I can see unassigned tickets and my own tickets, filtered by status and priority, so I know what to work on first.
- Given I filter by status Open, when the list loads, then I see only Open tickets
- Given tickets are assigned to other agents, when I view the list, then I do not see them

### US-7 (Admin): Manage agents
As an admin, I can add or deactivate agents, so the right people handle tickets.
- Given I am an admin, when I add an agent with a valid email, then they can log in as an agent
- Given an agent is deactivated, when they try to log in, then access is refused

### US-8 (Admin): See workload summary
As an admin, I can see ticket counts by status, so I know the workload.
- Given I open the summary, then I see the number of tickets for each status
- Given I am not an admin, when I request the summary, then I get 403

### US-9 (Admin): Manage categories
As an admin, I can create and rename ticket categories, so tickets are organised and easy to filter.
- Given I am an admin, when I create a category with a unique name, then it is available when creating tickets
- Given the name already exists, when I submit, then I see "Category already exists"