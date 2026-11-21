# Systems Analysis & Design (SAD)

> How to plan, analyse, and design an information system *before* writing code, so you build the right thing. (Based on Valacich & George, *Modern Systems Analysis and Design*, and Shelly & Rosenblatt, *Systems Analysis and Design*.)

**Topics:** [Systems Development & SDLC](#1-systems-development--the-sdlc) · [Methodologies (Waterfall, Agile, OO)](#2-development-methodologies) · [Identifying & Selecting Projects](#3-identifying--selecting-projects) · [Project Management](#4-project-management) · [Initiating & Planning (Feasibility)](#5-initiating--planning-feasibility) · [Requirements Determination](#6-requirements-determination) · [Process Modelling (DFDs)](#7-structured-process-modelling-dfds) · [Data Dictionary & Process Descriptions](#8-data-dictionary--process-description-tools) · [Practice: LBMS Example](#9-worked-example-library-management-system)

---

## 1. Systems Development & the SDLC

An **information system (IS)** = people + procedures + hardware + software + data, working together to support an organisation.
A **systems analyst** studies business problems and designs IS solutions. They bridge business users and technical developers.

> 🏗️ **Analogy:** a systems analyst is like an architect. Before anyone lays bricks (code), they talk to the family (users), learn how they live (requirements), and draw the plans (design).

### The Systems Development Life Cycle (SDLC)
| Phase | Key question | Main products |
|---|---|---|
| **1. Planning** (identify, initiate & plan) | Should we do this project? | priority list of projects, **Baseline Project Plan**, feasibility study |
| **2. Analysis** | What must the system do? | **requirements**, models of the current and new system (DFDs, ERDs), recommended solution |
| **3. Design** (logical & physical) | How will it work? | detailed specs: interfaces, database, programs, technology |
| **4. Implementation** | Build it and roll it out | code, testing, installation, documentation, training |
| **5. Maintenance** | Keep it working | fixes, enhancements |

- **Logical design:** independent of technology ("the system stores customer orders").
- **Physical design:** specific technology ("orders are stored in a MySQL table on AWS").

The SDLC can be drawn as a **circle** (when a system ends, a new cycle starts) or a **spiral** (iterating through the phases at deeper levels). The **analysis–design–code–test loop** is the iterative heart of development.

---

## 2. Development Methodologies

### Traditional waterfall
```
Planning → Analysis → Design → Implementation → Maintenance
   (each phase flows down; going back is hard)
```
- ✅ Clear structure, documentation, milestones
- ❌ Changes late in the project are **very expensive**; users see the system only at the end; requirements get "frozen" early

> 🌊 **Analogy:** water flowing down a waterfall can't flow back up.

### Evolutionary / prototyping
Build a quick working model, get user feedback, refine. Good when requirements are unclear.

### Agile methodologies
Three key principles: **adaptive** (not predictive), focus on **people** (not roles), **self-adaptive processes**.

**Agile Manifesto**, valuing:
- **Individuals and interactions** over processes and tools
- **Working software** over comprehensive documentation
- **Customer collaboration** over contract negotiation
- **Responding to change** over following a plan

| Method | Key practices |
|---|---|
| **eXtreme Programming (XP)** | short cycles, **pair programming**, test-first, continuous integration, simple design, on-site customer, coding standards |
| **Scrum** | **sprints** (2–4 weeks), **product backlog**, sprint backlog, **daily stand-up**, sprint review and retrospective; roles: **Product Owner**, **Scrum Master**, **Development Team** |

> 🏉 **Analogy:** Scrum is named after the rugby formation. The whole team moves the ball forward together in short bursts, adjusting as the game changes.

**When agile fits:** unclear or changing requirements, small co-located teams, customers available. **When waterfall fits:** stable requirements, regulatory/safety-critical systems, large fixed contracts.

### Object-Oriented Analysis & Design (OOAD) and RUP
Model the system as **objects** that combine data and behaviour, using **UML** diagrams (use case, class, sequence, activity).
**Rational Unified Process (RUP)**: iterative, with four phases: **Inception → Elaboration → Construction → Transition**.

---

## 3. Identifying & Selecting Projects

Organisations have more project ideas than resources, so they must choose.

**Sources of project ideas:** managers and business units, IS department, formal planning groups, external pressures (new laws, competitors).

**Selection process:**
1. **Identify** potential development projects.
2. **Classify and rank** them using evaluation criteria.
3. **Select** projects for development.

**Evaluation criteria:** value chain analysis, strategic alignment, potential benefits, resource availability, project size/duration, technical difficulty and risks.

> 🍕 **Analogy:** a student with RM50 and many wants (textbook, gaming headset, gym membership) ranks them by need, cost, and benefit before spending.

**Outcome:** a schedule of approved projects. Projects should align with the organisation's **Information Systems Plan (ISP)**, which links IS to the corporate strategic plan.

---

## 4. Project Management

A **project** = a planned, temporary set of activities to achieve a goal with a beginning and an end.
The **project manager** juggles: **customers, management, team members, resources, scope, time, cost, risk**.

### Project management phases
1. **Initiating:** establish the team, relationship with the customer, initiation plan, management procedures, project workbook.
2. **Planning:** scope, work breakdown, estimate resources, schedule, communication plan, standards, risk analysis, Statement of Work.
3. **Executing:** carry out the plan, monitor progress, manage changes, keep the workbook up to date, communicate status.
4. **Closing down:** close the project, post-project review, close the customer contract.

### Planning tools
| Tool | What it shows |
|---|---|
| **Work Breakdown Structure (WBS)** | the project broken into smaller, manageable tasks |
| **Gantt chart** | tasks as horizontal bars on a timeline (easy to see durations and overlaps) |
| **Network diagram (PERT/CPM)** | task dependencies; the **critical path** = the longest path, which sets the minimum project duration (any delay on it delays the whole project) |
| **Communication matrix** | who receives which reports/information and how often |

**PERT time estimate:** `ET = (optimistic + 4 × realistic + pessimistic) / 6`
*Example:* o = 2, r = 4, p = 12 days → `(2 + 16 + 12)/6 = 5 days`.

---

## 5. Initiating & Planning (Feasibility)

**Project initiation and planning** produces the **Baseline Project Plan (BPP)** and the **Statement of Work (SOW)**.

### Feasibility assessment
| Type | Question |
|---|---|
| **Economic** | Do the benefits outweigh the costs? |
| **Technical** | Do we have the technology and skills? What are the risks? |
| **Operational** | Will it solve the problem, and will people use it? |
| **Schedule** | Can it be done in time? |
| **Legal & contractual** | Any legal issues (copyright, data protection, contracts)? |
| **Political** | How do key stakeholders feel about it? |

### Economic feasibility (cost–benefit analysis)
- **Tangible benefits:** measurable (fewer errors, lower costs, more sales).
- **Intangible benefits:** hard to measure (better morale, customer goodwill, faster decisions).
- **One-time costs:** development, hardware, training, data conversion.
- **Recurring costs:** maintenance, licences, hosting, staff.

**Techniques** (accounting for the time value of money, since RM1 today is worth more than RM1 next year):
- **Net Present Value (NPV):** `PV = FV / (1 + i)ⁿ`. Sum the discounted benefits minus the discounted costs. NPV > 0 → worth doing.
- **Return on Investment (ROI):** `(total benefits − total costs) / total costs`.
- **Break-even analysis:** when do cumulative benefits catch up with cumulative costs? (payback period)

*Example:* a benefit of RM10,000 in 2 years at a 10% discount rate is worth `10000 / 1.1² ≈ RM8,264` today.

### Technical risk
Risk depends on **project size** (bigger = riskier), **project structure** (well-defined requirements = less risk), and the team's **familiarity with the technology** (new tech = riskier).

### Baseline Project Plan outline
1. Introduction (overview, recommendation)
2. System description (alternatives, recommended system)
3. Feasibility assessment (economic, technical, operational, legal, political, schedule)
4. Management issues (team, communication plan, standards)

---

## 6. Requirements Determination

**Requirements** = what the system must do (**functional**) and the qualities it must have (**non-functional**: performance, security, usability, reliability).

*Functional:* "The system shall let students register for courses."
*Non-functional:* "The page shall load within 2 seconds for 1,000 concurrent users."

> 🍰 **Analogy:** ordering a birthday cake. Functional = "chocolate, 2 tiers, says 'Happy 21st'". Non-functional = "ready by 5 PM, nut-free, survives a 30-minute drive".

**Good analyst traits:** impertinence (question everything), impartiality, relax constraints, attention to detail, reframing.

### Fact-finding techniques
| Technique | Pros | Cons |
|---|---|---|
| **Interviews** | rich detail, can follow up | time-consuming; interviewee bias |
| **Questionnaires / surveys** | reach many people cheaply | low response, no follow-up |
| **Observation** | see what people *really* do | people act differently when watched |
| **Document analysis** | forms, reports, procedures show the current system | may be outdated |
| **JAD** (Joint Application Design) | users, managers, and analysts in intensive workshops; fast consensus | expensive, needs a skilled facilitator |
| **Prototyping** | users react to something concrete | may raise unrealistic expectations |

**Interview tips:** prepare, use open-ended ("How do you handle late returns?") and closed-ended ("How many orders per day?") questions, listen, take notes, don't promise features, and send a summary afterwards.

**Radical methods:** Business Process Reengineering (BPR), which rethinks processes from scratch rather than automating the existing ones.

---

## 7. Structured Process Modelling (DFDs)

A **Data Flow Diagram (DFD)** shows **how data moves** through a system: where it comes from, how it's processed, where it's stored, and where it goes. It does **not** show program logic, timing, or decisions.

> 🚰 **Analogy:** a plumbing diagram for information. It shows the pipes (data flows), the machines (processes), and the tanks (data stores), not the exact steps each machine performs inside.

### DFD symbols
| Symbol | Gane & Sarson | Yourdon/DeMarco | Rules |
|---|---|---|---|
| **Process** | rounded rectangle | circle | named with a **verb + noun** ("Verify Order"); must have **at least one input and one output** |
| **Data flow** | arrow | arrow | named with a **noun** ("Order Details"); at least one end must be a process |
| **Data store** | open-ended rectangle | two parallel lines | named with a plural noun ("Students"); needs at least one flow in and one out |
| **External entity** (source/sink) | square | square | outside the system ("Customer", "Bank") |

### Common errors
- **Black hole:** a process with inputs but **no outputs**.
- **Miracle (spontaneous generation):** a process with outputs but **no inputs**.
- **Grey hole:** the inputs can't possibly produce the outputs.
- Data flowing directly between two stores, two entities, or an entity and a store (there must be a process in between).

### Levels of DFDs (decomposition)
| Level | Description |
|---|---|
| **Context diagram** | the whole system as **one process (0)**, with all external entities and the main data flows. No data stores. |
| **Level 0 (diagram 0)** | breaks the system into its major processes (1.0, 2.0, 3.0…) and shows the data stores |
| **Level 1, 2…** | break individual processes into sub-processes (1.1, 1.2…) |
| Primitive | lowest level, not decomposed further |

**Balancing:** the inputs and outputs of a parent process must match those of its child diagram (no new external flows appearing out of nowhere).

> 🔍 **Analogy:** zooming in on Google Maps. The context diagram is the country view, level 0 the states, level 1 the city streets. The roads leaving one view must match the roads leaving the zoomed-in view.

**Creating a set of DFDs:**
1. Draw the context diagram.
2. Draw diagram 0.
3. Draw lower-level diagrams as needed.
4. Check balancing and the rules.

---

## 8. Data Dictionary & Process Description Tools

### Data dictionary
A central catalogue (**metadata**) describing every element in the DFDs: data elements, data flows, data stores, processes, entities, records.
*Example entry:*
```
Data element: STUDENT_ID
  Type: Alphanumeric   Length: 10   Format: U2xxxxxxx
  Description: Unique identifier assigned at enrolment
  Source: Registration form   Validation: must be unique, not null
```

### Process description tools (what happens *inside* a primitive process)
**Structured English:** plain English with programming-like structure.
```
FOR each book returned
    IF return_date > due_date
        CALCULATE fine = (return_date - due_date) × RM0.50
        ADD fine to member account
    ENDIF
    UPDATE book status to "Available"
ENDFOR
```

**Decision table:** shows actions for every combination of conditions.
| Conditions / Actions | R1 | R2 | R3 | R4 |
|---|---|---|---|---|
| Member? | Y | Y | N | N |
| Order > RM100? | Y | N | Y | N |
| **Free shipping** | ✔ | | ✔ | |
| **10% discount** | ✔ | ✔ | | |

With n conditions there are `2ⁿ` rules. Simplify by merging rules where a condition doesn't matter ("–").

**Decision tree:** the same logic drawn as branches, easier to read for a few conditions.

> 🧭 **When to use which:** structured English for sequences and loops; decision tables for many combinations; decision trees for simple branching that non-technical users must read.

---

## 9. Worked Example: Library Management System

*(Matches the LBMS practice exercise in the notes.)*

**Entities:** Member, Librarian, Supplier (external); data stores: Members, Books, Loans, Fines.

**Context diagram (level 0 overview):**
```
            membership details          
 MEMBER ────────────────────────► ┌──────────────────┐ ◄──── book orders ──── SUPPLIER
        ◄──── loan receipt ─────── │ 0  Library Mgmt  │ ───── purchase order ──►
        ◄──── fine notice ──────── │     System       │
 LIBRARIAN ── book details ──────► └──────────────────┘ ──── reports ────► LIBRARIAN
```

**Diagram 0 (major processes):**
1. **Register Member** (Member → member details → *Members* store)
2. **Borrow Book** (Member's request + *Books* + *Members* → *Loans*, receipt to Member)
3. **Return Book** (returned book → updates *Loans* and *Books*; if late → *Fines*)
4. **Manage Fines** (*Fines* → fine notice to Member; payment → update)
5. **Order Books** (*Books* low stock → purchase order to Supplier)
6. **Generate Reports** (*Loans*, *Fines*, *Books* → reports to Librarian)

Check: every process has inputs and outputs ✅, every store is read and written ✅, flows to and from entities match the context diagram ✅.

---

### ✅ Quick self-check
1. Name the five SDLC phases. *(planning, analysis, design, implementation, maintenance)*
2. What's the main weakness of waterfall? *(expensive to change late; users see the result late)*
3. Three roles in Scrum? *(Product Owner, Scrum Master, Development Team)*
4. A process with inputs but no outputs is called…? *(a black hole)*
5. What does "balancing" mean in DFDs? *(parent and child diagrams have the same inputs and outputs)*
6. NPV > 0 means…? *(benefits outweigh costs after discounting; economically feasible)*
