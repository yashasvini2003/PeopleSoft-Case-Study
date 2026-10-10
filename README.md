# PeopleSoft-Case-Study


# Oracle PeopleSoft HCM Absence Management

**A Business Analyst Case Study: Vacation Requests, Delegated Approval, and Paid/Unpaid Absence**

## Overview

Asha Shah works as an Operations Coordinator at Northbridge Services, a fictional company in Toronto. She has been with the company for eighteen months and helps coordinate client schedules and day-to-day team activities. For this scenario, Northbridge uses Oracle PeopleSoft HCM Absence Management to handle employee leave requests, approval routing, and entitlement processing. Her sister is getting married outside Toronto on 17 November 2026, and Asha would like time to travel, attend the wedding, and return home. She plans to request vacation leave from Monday, 16 November, to Wednesday, 18 November: three working days.

Before submitting her request, Asha checks how much paid vacation she has available. She expects to have sixteen hours by the time her leave starts, but her three-day request covers twenty-four working hours. This raises a practical question: can she take all three days, and would the last day be unpaid? She needs a clear answer before making travel arrangements.

There is another complication. Her manager, Daniel Patel, will be away when the request is reviewed. Daniel has arranged for Elena Garcia to review eligible absence requests during his absence, but the delegation must cover the correct transaction, be accepted, and remain valid when Elena makes the decision. If that arrangement is missing or invalid, someone must resolve the approval route rather than leave Asha wondering what happened to her request.

This case study follows Asha's request through employee self-service, balance forecasting, delegated review, and the three possible review outcomes: approval, denial, or return for clarification. It then looks at what an approved absence would mean for her leave balance and an illustrative November gross-pay calculation.

The analysis follows the request through PeopleSoft and asks what a business analyst would need to clarify at each stage: required employee information, forecast rules, approval authority, exceptional routes, and the handoff to absence processing and payroll. The requirements, user stories, swimlane diagram, and SQL examples explain those decisions.

> **Project context:** This is an independent learning case study informed by Oracle documentation and my Software Development background at Seneca College, now Seneca Polytechnic. Northbridge, its employees, and its policies are fictional. The work demonstrates research and analysis; it does not represent employment experience or an implemented PeopleSoft solution.

## Contents

- [The business problem](#the-business-problem)
- [People and responsibilities](#people-and-responsibilities)
- [Questions I would ask before agreeing the process](#questions-i-would-ask-before-agreeing-the-process)
- [Scenario facts and policy assumptions](#scenario-facts-and-policy-assumptions)
- [Following Asha's request](#following-ashas-request)
- [Business requirements](#business-requirements)
- [User stories and acceptance criteria](#user-stories-and-acceptance-criteria)
- [Swimlane diagram](#swimlane-diagram)
- [Entitlement and monthly calculations](#entitlement-and-monthly-calculations)
- [Configuration choices, quality, and access](#configuration-choices-quality-and-access)
- [SQL reporting exercise](#sql-reporting-exercise)
- [Proposed user acceptance testing](#proposed-user-acceptance-testing)
- [Business analysis techniques and academic connections](#business-analysis-techniques-and-academic-connections)
- [Coverage of the thirteen learning topics](#coverage-of-the-thirteen-learning-topics)
- [Presenting the backlog in GitHub](#presenting-the-backlog-in-github)
- [What I learned and what I would explore next](#what-i-learned-and-what-i-would-explore-next)
- [References](#references)

## The business problem

For Asha, success means knowing whether she can take the requested time and understanding its expected pay treatment. For Northbridge, success means reaching an accountable decision, keeping enough staff available, and passing accurate absence information to payroll.

For this exercise, I assume Northbridge wants to address three weaknesses in its leave process: employees receive unclear balance information, requests can wait without an identifiable approver, and payroll needs a clearer explanation of paid and unpaid hours. These are scenario assumptions rather than findings from interviews with a real company.

The proposed outcome is a process in which:

- Employees can enter a valid absence request and understand the expected paid/unpaid split before proceeding.
- A manager or an authorized delegate can review the request and explain the decision.
- Missing approval ownership is visible and has a responsible administrator.
- Only eligible, approved events move into absence processing.
- HR and payroll can reconcile the request, entitlement usage, and calculated hours.

The scope is PeopleSoft HCM Absence Management and its connections to approval, delegation, security, and reporting. FSCM and Campus Solutions provide product-suite context but are outside this scenario. This repository contains analysis in one README; it does not include an application.

## People and responsibilities

| Person or role | What matters to them | Responsibility in this case |
| --- | --- | --- |
| Asha â employee | Time for the wedding and clear pay expectations | Enter accurate details, review the forecast, submit, and respond to clarification |
| Daniel â manager | Team coverage and consistent decisions | Review when available; establish an appropriate delegation while away |
| Elena â delegate | Clear authority and the information needed to decide | Accept the delegation and act only within its valid scope and period |
| HR policy owner | Consistent entitlement and absence treatment | Confirm eligibility, accrual, unpaid treatment, and cancellation rules |
| Approval administrator | Requests with a valid, accountable approval route | Investigate exceptions and perform authorized route repair |
| Payroll specialist | Accurate hours and explainable gross pay | Process eligible events and reconcile paid/unpaid units with worked time |
| PeopleSoft functional specialist | A workable relationship between policy and configuration | Assess delivered capability, configuration choices, and gaps |
| Security/reporting owners | Appropriate access to employee and payroll information | Confirm function permissions, data scope, and report access |
| Business analyst | Shared understanding of the problem and expected behavior | Document assumptions, questions, requirements, stories, and acceptance scenarios |

Being an HR employee does not automatically make someone Asha's substitute approver. Repairing a route and approving a request are separate responsibilities.

## Questions I would ask before agreeing the process

The documentation helped me identify capabilities, but it cannot decide Northbridge's employment policies. In a real project, I would use these questions to prepare discussions with the relevant stakeholders. No interviews were conducted for this exercise.

| Question | Who should confirm it? | Why it changes the requirements |
| --- | --- | --- |
| Which employees qualify for this vacation entitlement, and when do credits become usable? | HR | Determines the balance available on each absence date |
| Are balances measured in hours or days? How are part-time schedules and partial days handled? | HR and functional specialist | Prevents calendar-day counting from producing incorrect units |
| Are holidays and non-working days excluded from the requested duration? | HR and payroll | Changes requested hours and monthly scheduled hours |
| What happens when a request exceeds available entitlement: unpaid time, borrowing, redirection, or rejection? | HR | Determines the forecast and processing rules |
| Does the employee need to agree to unpaid time, and what happens if that estimate changes? | HR, payroll, and functional specialist | Defines the acknowledgement and reforecast controls |
| Who may approve while a manager is away, and what if the delegation expires before a decision? | Manager, security owner, and approval administrator | Defines valid authority and exception handling |
| Which approval actions are supported at each stage? | Functional specialist and business owner | Prevents assuming every Push Back returns to the employee |
| What may be cancelled before processing, and how are processed events corrected? | HR and payroll | Defines cancellation and recalculation boundaries |
| Which event statuses does forecasting include, and how are competing requests handled? | Functional specialist | Prevents stale balances and double counting |
| What are payroll cutoffs, and what evidence confirms actual worked and absent hours? | Payroll | Separates approval from the final pay result |
| Which request details, comments, reports, and pay figures may each role see? | Security and privacy owners | Defines access beyond whether a tile is visible |
| How will we measure improvement? | Process owner | Establishes a baseline for approval time, exceptions, and payroll corrections |

I have not invented an improvement percentage or approval-time target. Those would need a baseline and agreement with the process owner.

## Scenario facts and policy assumptions

The following choices make the example calculable. They are Northbridge's fictional rules for this study, not universal PeopleSoft defaults or payroll guidance.

| Item | Assumption |
| --- | --- |
| Employee | Asha Shah, E1001, Operations Coordinator |
| Absence name used for the request | Vacation |
| Reason | Family event: attending her sister's wedding and travelling |
| Absence dates | 16â18 November 2026 |
| Submission and example review | Submitted 10 November; reviewed 12 November |
| Work schedule | MondayâFriday, 8 hours per working day |
| Demonstration calendar | No holiday exclusions; weekends have zero scheduled hours |
| Opening balance before November's credit | 8 hours |
| Monthly credit | 8 hours, usable on the first day of the month for an eligible employee |
| November credit | 8 hours effective 1 November |
| Available for the main request | 16 hours; no other events consume this balance |
| Shortfall treatment | Approved excess may be unpaid with the employee's agreement |
| Borrowing or negative balance | Neither is allowed in this example |
| Approval | One authorized approval step; manager or valid accepted delegate |
| Daniel's delegation to Elena | Absence approvals, 9â20 November; accepted before submission |
| Illustrative hourly rate | CAD 30; gross-pay demonstration only |

Selecting Vacation is a scenario choice. A family wedding is the reason for this request; the case does not assume that a separate special family-leave entitlement exists.

The monthly-credit policy would produce 12 Ã 8 = **96 hours**, or **12 eight-hour days**, over twelve eligible months. That is a mathematical description of the assumed policy. Eligibility, carryover, joiner proration, waiting periods, schedule changes, and holiday treatment would need separate decisions in a real project.

### Terms I use in the case

| Term | Meaning here |
| --- | --- |
| Absence request | The employee's submitted request for specified absence dates and details |
| Absence type and reason | The configured category and reason used to classify an absence |
| Absence name / take | The selected absence and the configured rules governing how it is evaluated and consumes entitlement |
| Entitlement | The eligible leave units available under the agreed rules |
| Forecast | An estimate using the candidate event, relevant existing events, calendars, and configured rules |
| Approval | Permission to take the absence; it does not prove that payroll processing has finished |
| Absence processing | Calculation of eligible absence and entitlement results under the configured rules |
| Delegate / proxy | A person with accepted delegated authority for the relevant transaction and period |

Oracle distinguishes these concepts in its Absence Management documentation. The relationships informed this model; the selected numbers and company policies are my scenario assumptions. See references O06âO12.

## Following Asha's request

### 1. Preparing and submitting the request

On 10 November, Asha signs in through Northbridge's browser-based employee access. She opens the configured absence function, selects Vacation, enters 16â18 November, and provides the required reason. The example navigation is Employee Self Service â Time â Manage Absences; the actual tiles and route depend on the organization's setup. [O03, O04, O07]

The proposed process checks her eligibility, required information, date order, and schedule. Three working days become twenty-four hours. If a required field is missing or a date is invalid, Asha receives a message explaining what to correct before submission.

She then reviews the forecast: sixteen paid hours and eight unpaid hours. Asha decides to proceed because attending the wedding is important to her and she understands the expected unpaid day. The case proposes recording that acknowledgement before submission. Whether this requires configuration, an additional control, or an extension needs a fit-gap review; I am not assuming a delivered PeopleSoft checkbox.

After submission, Asha receives an identifiable request and can track its status. Forecasting evaluates the candidate once alongside the agreed population of existing events. The model includes relevant approved events; a real implementation must confirm the configured status filters and handling of pending requests. [O10]

### 2. Finding an authorized reviewer

Daniel is away, so the process evaluates his accepted delegation to Elena. It checks the covered transaction, effective dates, delegate eligibility, and current authority. Elena can review the request on 12 November because the scenario's delegation is valid for absence approvals on that date. [O13]

If Daniel had not arranged an eligible delegation, or Elena had rejected it, the case would follow an exception route. An authorized approval administrator would investigate, identify an appropriate approver, record the reason for reassignment, and validate the repaired route. The request would remain unresolved until that work was complete. Northbridge's chosen process does not automatically approve a request because its manager is unavailable.

Oracle's approval documentation describes identifying approvers from the configured approval definition and notifying an approval administrator when the relevant steps cannot identify an approver. The exception handling here applies that capability to the selected one-step policy. [O08, O08A]

### 3. Reviewing the request and reaching an outcome

Elena considers the requested dates, team coverage, and permitted request information. The proposed control refreshes the forecast before approval. If another approved event has reduced Asha's balance, Asha should see the changed split and make a new decision before an approval relies on it. This concurrency and acknowledgement behavior must be assessed with the functional specialist.

| Review outcome | What happens to the request? | What does Asha do next? | Absence/pay consequence |
| --- | --- | --- | --- |
| **Approve** | An authorized decision is recorded and communicated | Take the authorized absence, assuming her plans remain unchanged | Eligible processing applies the agreed rules; the main case becomes 16 paid and 8 unpaid hours |
| **Deny** | The process ends with an explanation | Review the reason and discuss other dates if needed | This request contributes no approved absence allocation; denial itself does not deduct pay |
| **Push Back** | At the supported first approval step, it returns for attention with a comment | Clarify or correct the request and resubmit through validation and forecasting | No approved absence allocation yet; the eventual pay outcome remains unresolved |

An example approval comment is: âCoverage is available for these dates. Approved with the acknowledged expected allocation of sixteen paid hours and eight unpaid hours.â An example denial reason is that the requested dates cannot be supported because essential team coverage is unavailable. An example Push Back comment asks Asha to clarify a mismatch between the selected full days and her request notes.

Push Back is not interchangeable with denial. In this case, it means the first reviewer needs a correctable detail clarified. At a later approval stage, a supported pushback can return to a previous approver instead. The available actions and exact labels must be checked for the transaction and deployed configuration. **Submitted, Approved, Denied, Needs Rework, and Cancelled are conceptual labels in this study**, not asserted PeopleSoft database codes. [O08, O08A]

### 4. After approval

Approval allows the absence to proceed, but final calculation remains a separate activity. Payroll processes the eligible event using the agreed take and entitlement rules, reconciles the absence with actual worked time, and explains the resulting gross amount. The forecast remains an estimate until the relevant event data has been processed. [O11, O12]

If Asha's plans change, cancellation follows the applicable authority and processing-stage rules. A permitted cancellation before processing excludes the event from future allocation. An event already processed or finalized requires the authorized correction and recalculation procedure rather than silent deletion. [O09]

## Business requirements

These describe the outcomes Northbridge needs. The priorities are provisional for the study: **Must** is necessary for the core process; **Should** is useful supporting behavior.

| ID | Business requirement | Priority |
| --- | --- | --- |
| BR01 | Requested duration must follow the employee's agreed working schedule and unit. | Must |
| BR02 | Entitlement must become usable according to eligibility and effective credit dates. | Must |
| BR03 | Paid absence must not exceed available entitlement; this case permits no future borrowing or negative balance. | Must |
| BR04 | The employee must understand and agree to the expected unpaid component before approval relies on it. | Must |
| BR05 | Every approval decision must have an identifiable, authorized decision-maker. | Must |
| BR06 | A missing approver must create a visible exception with authorized resolution. | Must |
| BR07 | Approval, denial, and supported rework must be distinct and explained to the requester. | Must |
| BR08 | Submitted, denied, rework, and cancelled requests must stay outside approved absence processing. | Must |
| BR09 | Requested hours, paid/unpaid hours, and entitlement consumption must reconcile without duplication. | Must |
| BR10 | The monthly gross calculation must use verified worked time and avoid deducting unpaid hours twice. | Must |
| BR11 | Cancellation and correction must respect authority and processing boundaries. | Must |
| BR12 | Reports must define their population, period, status meaning, and authorized audience. | Should |

## User stories and acceptance criteria

The stories below turn those requirements into behavior that stakeholders could review. **FR01âFR18** identify the functional behavior represented by the matching story; **U01âU26** refer to the acceptance scenarios later in this README. These are proposed requirements, not features implemented in PeopleSoft.

### US01 â Request time for a family event

**As an employee, I want to request vacation for my sister's wedding so that I can arrange the time away and receive a clear decision.**

- Given Asha's eight-hour weekday schedule, when she selects 16â18 November, then the requested duration is twenty-four working hours.
- Before proceeding, she can understand whether all requested hours are expected to be paid.

**Trace:** FR01 Â· BR01âBR02 Â· U01 Â· Must

### US02 â Access my absence information securely

**As an employee, I want to open my own absence tasks so that I can manage my requests securely.**

- An authenticated, authorized employee can reach the configured absence function.
- Asha cannot view another employee's record by changing a request identifier or using a direct link.

**Trace:** FR02 Â· BR05, BR12 Â· U02 Â· Must

### US03 â Enter a valid request

**As an employee, I want to enter the absence name, dates, and required details so that the request accurately describes the time I need.**

- Missing required information, ineligibility, or an end date before the start date prevents valid submission and explains the correction.
- Duration follows the schedule and configured units, including non-working days and any supported partial days.

**Trace:** FR03 Â· BR01âBR02 Â· U01, U03âU04, U21 Â· Must

### US04 â Understand the expected paid and unpaid hours

**As an employee, I want to review the expected pay treatment before submitting so that I can make an informed decision.**

- Asha's main forecast shows twenty-four requested hours: sixteen paid and eight unpaid.
- If she declines the proposed unpaid treatment, she can revise or withdraw the candidate rather than submit on that basis.
- If the split changes before approval, the proposed control requires renewed acknowledgement. Delivered fit remains to be assessed.

**Trace:** FR04 Â· BR03âBR04 Â· U05âU06, U20 Â· Must

### US05 â Submit and identify my request

**As an employee, I want to submit a validated request and keep its identifier so that I can follow its progress.**

- A successful submission creates one identifiable request with a visible submitted state.
- Repeated clicks do not create duplicate business submissions.
- An unresolved route remains visible as an exception and is not presented as approval.

**Trace:** FR05 Â· BR05âBR06 Â· U07, U10 Â· Must

### US06 â Delegate approval while I am away

**As a manager, I want to delegate eligible absence approvals for a defined period so that employees can receive decisions while I am unavailable.**

- Daniel specifies the eligible delegate, transaction scope, and dates; Elena must accept the delegation.
- Rejected, revoked, expired, or unrelated delegation cannot authorize a decision.
- Authority is checked when the decision is made, including when delegation expires after routing.

**Trace:** FR06 Â· BR05 Â· U08âU09, U22 Â· Must

### US07 â Resolve an approval-routing exception

**As an approval administrator, I want to identify and repair an unresolved route so that each request has an accountable reviewer.**

- A missing eligible approver produces an actionable exception rather than automatic approval.
- Reassignment uses an authorized administrator and valid recipient, with actor, time, and reason recorded.
- Administrative access alone does not grant approval or payroll authority.

**Trace:** FR07 Â· BR05âBR06 Â· U09âU10, U22 Â· Must

### US08 â Approve a suitable request

**As an authorized manager or delegate, I want to approve a suitable request so that the employee has permission to take the specified absence.**

- The reviewer can assess permitted request information and the current expected allocation.
- Approval checks current authority and the agreed policy/acknowledgement conditions; the decision is recorded and communicated.
- Approved remains distinguishable from absence-processed or payroll-finalized.

**Trace:** FR08 Â· BR04âBR05, BR07âBR09 Â· U11, U20, U23 Â· Must

### US09 â Deny with an explanation

**As an authorized reviewer, I want to deny a request with a reason so that the employee understands why it cannot proceed.**

- Under the chosen policy, denial records a reason and communicates the decision.
- The denied request creates no approved paid or unpaid absence allocation.
- If the employee works the scheduled hours, pay follows worked-time rules; denial is not itself a pay deduction.

**Trace:** FR09 Â· BR07âBR08 Â· U12 Â· Must

### US10 â Return a request for clarification

**As the authorized first reviewer, I want to push back a correctable request so that the employee can clarify it before a final decision.**

- The transaction and approval stage must support the selected action; a comment explains what needs attention.
- The conceptual Needs Rework request is excluded from approved processing.
- Correction and resubmission repeat validation, forecasting, and route checks and preserve decision history.
- A later-stage pushback follows its configured route rather than always returning to the employee.

**Trace:** FR10 Â· BR07âBR08 Â· U13âU14, U26 Â· Must

### US11 â Track the outcome and next action

**As an employee, I want to see my request status and decision comments so that I know whether to wait, revise, or act on a decision.**

- Submitted, Approved, Denied, and Needs Rework are distinguishable in the proposed view.
- Messages identify the next action and any outstanding route issue without exposing another person's protected information.

**Trace:** FR11 Â· BR06âBR07 Â· U11âU14 Â· Must

### US12 â Cancel when my plans change

**As an employee, I want to request cancellation when permitted so that my absence record reflects my updated plans.**

- Cancellation checks employee authority, the required reason, and the event's processing stage.
- A valid pre-processing cancellation prevents future allocation from that event.
- A processed or finalized event uses a controlled correction procedure and any required recalculation.

**Trace:** FR12 Â· BR08, BR11 Â· U15âU16 Â· Must

### US13 â Apply entitlement on the correct date

**As the HR policy owner, I want monthly credits to become usable on their agreed dates so that leave availability follows the policy.**

- The demonstration credit is eight hours on the first day of each eligible month and is not duplicated.
- A credit effective in December cannot pay an absence in November under this no-borrowing policy.

**Trace:** FR13 Â· BR02âBR03 Â· U17, U24 Â· Must

### US14 â Process approved absence hours accurately

**As a payroll specialist, I want eligible approved hours allocated under the agreed rules so that paid time, unpaid time, and balances reconcile.**

- Asha's approved event allocates eight paid hours on 16 November, eight paid hours on 17 November, and eight unpaid hours on 18 November.
- Paid plus unpaid equals requested hours; the closing entitlement is zero, with no negative balance.
- Restarting processing does not consume entitlement or create paid units twice.

**Trace:** FR14 Â· BR03, BR08âBR09 Â· U18, U23âU24 Â· Must

### US15 â Explain the monthly gross amount

**As a payroll specialist, I want to reconcile worked and absence hours so that the illustrative monthly gross amount is explainable.**

- In the main branch, 144 worked hours plus sixteen paid absence hours produce 160 payable hours.
- At CAD 30 per hour, gross pay is CAD 4,800; the eight unpaid hours are not deducted a second time.
- Worked-time inputs are verified separately rather than inferred from a request's approval status.

**Trace:** FR15 Â· BR09âBR10 Â· U19 Â· Must

### US16 â Report outstanding work and calculated outcomes

**As an authorized reporting analyst, I want separate request-status and allocation reports so that I can distinguish outstanding decisions from processed absence results.**

- Reports identify pending ownership or an unresolved routing exception.
- Denied and rework requests remain visible in status reporting but outside approved allocations.
- Production reports require confirmed PeopleSoft records, approved Query access, and the applicable data restrictions.

**Trace:** FR16 Â· BR12 Â· U25 Â· Should

### US17 â Check the fit between requirements and configuration

**As a functional specialist, I want to assess the agreed requirements against PeopleSoft capability so that configuration choices and gaps are visible before delivery.**

- The review covers units, schedules, reasons, entitlement/take links, forecasting, approvals, delegation, notifications, and cancellation.
- Employee acknowledgement, forecast refresh, and concurrency controls receive an explicit fit-gap decision.
- A configuration is not described as implemented without environment evidence.

**Trace:** FR17 Â· BR01âBR12 Â· Relevant U01âU26 Â· Must

### US18 â Protect employee and payroll data

**As the security owner, I want function and record access limited to authorized people so that absence and pay information stay within the agreed scope.**

- Employees access their own records; reviewers access the valid team or delegated scope.
- Approval administration does not automatically grant access to individual gross pay.
- Access tests include direct links, altered identifiers, and exports, not only visible or hidden tiles.

**Trace:** FR18 Â· BR05, BR12 Â· U02, U25 Â· Must

## Swimlane diagram

![PeopleSoft HCM absence management swimlane diagram](swimlane-diagram.png)

The five lanes show responsibilities across the employee, PeopleSoft application, manager or authorized delegate, HR/approval administrator, and payroll/reporting. Rectangles show activities and diamonds show decisions. Read the main flow from top to bottom; arrows crossing a lane boundary show a handoff.

Green highlights approval and calculated paid units, red highlights denial or a blocked path, and amber highlights clarification or rework. Dashed arrows show supporting setup, optional cancellation, or status reporting. Approval, denial, and Push Back are alternative review outcomes.

The diagram follows Asha's vacation request and includes validation, dated entitlement, forecasting, unpaid-time acknowledgement, delegation checks, route repair, renewed acknowledgement when a forecast changes, absence processing, monthly gross reconciliation, and cancellation/correction.

The case uses a supported first-step Push Back route to the employee. At a later approval stage, pushback may return to a previous approver. Employee acknowledgement and forecast-refresh controls are proposed requirements subject to a configuration fit-gap review.

## Entitlement and monthly calculations

### Why two days are paid and one is unpaid

For an approved request under the selected rules:

| Calculation | Result |
| --- | ---: |
| Requested hours: 3 working days Ã 8 | 24 hours |
| Available entitlement: 8 opening + 8 November credit | 16 hours |
| Paid absence: minimum of requested and available hours | 16 hours |
| Unpaid absence: 24 â 16 | 8 hours |
| Closing entitlement: 16 â 16 | 0 hours |

The request is allocated in date order. The no-borrowing rule prevents a later monthly credit from being used early.

| Absence date | Available before day | Requested | Paid | Unpaid | Closing |
| --- | ---: | ---: | ---: | ---: | ---: |
| Monday, 16 November | 16 h | 8 h | 8 h | 0 h | 8 h |
| Tuesday, 17 November | 8 h | 8 h | 8 h | 0 h | 0 h |
| Wednesday, 18 November | 0 h | 8 h | 0 h | 8 h | 0 h |
| **Total** | â | **24 h** | **16 h** | **8 h** | **0 h** |

Oracle describes configurable handling of insufficient entitlement. Unpaid excess is the option selected for this scenario; other organizations can choose different treatment. [O11]

### Illustrative November gross pay

November 2026 contains twenty-one weekdays in the demonstration calendar: 21 Ã 8 = **168 scheduled hours**. Assume Asha takes the three approved days and works all other scheduled hours.

| Component | Hours | Calculation | Amount |
| --- | ---: | --- | ---: |
| Worked time | 144 | 18 days Ã 8 h Ã CAD 30 | CAD 4,320 |
| Paid vacation | 16 | 16 h Ã CAD 30 | CAD 480 |
| Unpaid absence | 8 | Zero payable amount | CAD 0 |
| **Gross total** | **160 payable** | **(144 + 16) Ã CAD 30** | **CAD 4,800** |

If all 168 scheduled hours were payable, the reference gross would be CAD 5,040. The difference is **CAD 240**, equal to eight unpaid hours at CAD 30.

The two equivalent calculations are `(worked hours + paid absence hours) Ã hourly rate` and, under these specific assumptions, `scheduled gross â unpaid hours Ã hourly rate`. Subtracting unpaid time from the first calculation again would double-count the reduction.

This is an hourly gross-pay illustration. It excludes taxes, benefits, overtime, salary proration, and holiday adjustments; it is not a calculation of take-home pay or a PeopleSoft payroll formula.

### What each review outcome means for pay

| Alternative | Approved absence from this request | Pay conclusion supported by the example |
| --- | --- | --- |
| Approved; Asha takes the leave | 16 paid / 8 unpaid hours after eligible processing | CAD 4,800 if the other 144 scheduled hours are worked |
| Denied; Asha works all scheduled hours | None | CAD 5,040; denial alone causes no deduction |
| Needs Rework | None yet | Final pay cannot be concluded from this unresolved state |
| Cancelled before processing; all scheduled hours worked | None | CAD 5,040 under the same rate and schedule |

If Asha were absent despite denial, the actual attendance and authorized absence classification would need separate resolution. The case does not assume either attendance or an automatic wage deduction from a denied request.

### A fully paid comparison

If an eligible employee began November with sixteen hours and received the eight-hour credit, twenty-four hours would be available. The same approved three-day vacation would be **twenty-four paid hours and zero unpaid hours**. With the same schedule, rate, and other worked time, November gross would be CAD 5,040. This comparison shows that requesting three days does not itself determine how many days are paid.

### A request crossing two months

Consider another employee, Sam Chen, requesting 30 Novemberâ2 December. Sam has no entitlement remaining before 30 November and receives an eight-hour credit effective 1 December.

| Date | New usable credit | Paid | Unpaid | Closing balance |
| --- | ---: | ---: | ---: | ---: |
| 30 November | 0 h | 0 h | 8 h | 0 h |
| 1 December | 8 h | 8 h | 0 h | 0 h |
| 2 December | 0 h | 0 h | 8 h | 0 h |
| **Total** | **8 h** | **8 h** | **16 h** | **0 h** |

November receives **zero paid/eight unpaid hours** from this request; December receives **eight paid/eight unpaid hours**. The December credit cannot pay November's absence under the chosen no-borrowing rule. A full December pay calculation would require all December worked time and other payable events, so this example reports the request's allocation rather than inventing a monthly total.

## Configuration choices, quality, and access

### Fit-gap questions

| Requirement area | Documentation connection | What still needs confirmation |
| --- | --- | --- |
| Request details and units | Self-service rules for the take [O09] | Required reasons, date restrictions, partial-day support, and employee schedule |
| Entitlement and unpaid excess | Entitlement/take concepts and take features [O06âO06C, O11] | Eligibility, credit timing, non-negative balance, and exact unpaid handling |
| Forecasting | Forecasting and balance inquiry [O10] | Calendars, event-status filters, and competing requests |
| Approval and Push Back | Approval Framework and Fluid approvals [O08, O08A] | Route, stage-specific actions, labels, notifications, and exception ownership |
| Delegation | Self-service delegation [O13] | Covered transaction, acceptance, eligible delegate, and decision-time authority |
| Unpaid acknowledgement and refreshed forecast | Proposed business controls | Delivered fit, configuration, process control, or extension; no implementation claim |
| Cancellation/correction | Self-service cancellation rules [O09] | Permissions and stage-specific correction/recalculation process |
| Reporting | PeopleSoft Query [O14] | Actual records, field meanings, period definitions, and report/security access |

### Proposed quality requirements

- **NFR01 â Data protection:** restrict functions and records to the approved role and employee scope, including export paths.
- **NFR02 â Accessibility:** core request and review tasks should support keyboard use and applicable assistive technology; validate in the target environment.
- **NFR03 â Auditability:** preserve actor, timestamp, reason, state changes, and request version for decisions and route repair.
- **NFR04 â Repeatability:** duplicate submissions, repeated decisions, or restarted processing must not duplicate business effects.
- **NFR05 â Clear messages:** explain the issue and next action in language the employee can understand.

| Role | Request visibility | Decision/processing authority | Individual gross-pay access |
| --- | --- | --- | --- |
| Employee | Own requests | Submit, respond, or request permitted cancellation | Own permitted payslip access only |
| Manager / accepted delegate | Authorized team or delegated scope | Assigned, currently valid review authority | Not granted by this review role |
| Approval administrator | Authorized exception scope | Route repair; approval only if separately authorized | Not granted by administration alone |
| Payroll specialist | Authorized processing population | Absence/payroll processing within scope | Authorized payroll scope |
| Reporting analyst | Approved report population | Reporting, not approval | Only if explicitly authorized |

Roles and permission lists support functional access, while record access requires the appropriate data restrictions as well. This matrix is a requirement for review, not a configuration established by the sample SQL. [O05, O05A]

## SQL reporting exercise

I included SQL because a BA should be able to explain what a report measures and check whether its numbers reconcile. The examples below use **synthetic SQLite tables and data**, not actual PeopleSoft records. They demonstrate joins, aggregation, date-based grouping, status filters, and explicit worked-time inputs. They do not reproduce PeopleSoft Query configuration or calculate entitlement automatically. [O14]

The allocation rows are sample processed results entered for checking reports. The arithmetic in the previous section explains how those values were derived. Separate employees represent alternative test outcomes so their hours are not combined with Asha's.

### Sample data definitions

| Table | Purpose |
| --- | --- |
| `employees` | Synthetic employee identifier, name, and illustrative rate |
| `work_calendar` | Demonstration scheduled hours by date |
| `absence_requests` | Request details, conceptual state, and approval ownership |
| `daily_allocations` | Sample paid/unpaid results by request and absence date |
| `worked_time_month` | Explicit worked-time input for the gross-pay example |
| `delegations` | Synthetic transaction scope, dates, and acceptance status |

#### SQL setup â Create and populate the sample database

The following setup and reports can be read or run together in an empty SQLite database. No software installation or PeopleSoft environment is needed to understand the examples.

```sql
PRAGMA foreign_keys = ON;

CREATE TABLE employees (
  employee_id TEXT PRIMARY KEY,
  employee_name TEXT NOT NULL,
  hourly_rate NUMERIC NOT NULL CHECK (hourly_rate >= 0)
);
CREATE TABLE work_calendar (
  work_date TEXT PRIMARY KEY,
  scheduled_hours NUMERIC NOT NULL CHECK (scheduled_hours >= 0)
);
CREATE TABLE absence_requests (
  request_id TEXT PRIMARY KEY,
  employee_id TEXT NOT NULL REFERENCES employees(employee_id),
  start_date TEXT NOT NULL,
  end_date TEXT NOT NULL CHECK (end_date >= start_date),
  conceptual_status TEXT NOT NULL,
  assigned_approver TEXT,
  routing_exception INTEGER NOT NULL CHECK (routing_exception IN (0, 1))
);
CREATE TABLE daily_allocations (
  request_id TEXT NOT NULL REFERENCES absence_requests(request_id),
  absence_date TEXT NOT NULL REFERENCES work_calendar(work_date),
  paid_hours NUMERIC NOT NULL CHECK (paid_hours >= 0),
  unpaid_hours NUMERIC NOT NULL CHECK (unpaid_hours >= 0),
  PRIMARY KEY (request_id, absence_date)
);
CREATE TABLE worked_time_month (
  employee_id TEXT NOT NULL REFERENCES employees(employee_id),
  pay_month TEXT NOT NULL,
  verified_worked_hours NUMERIC NOT NULL CHECK (verified_worked_hours >= 0),
  PRIMARY KEY (employee_id, pay_month)
);
CREATE TABLE delegations (
  delegation_id TEXT PRIMARY KEY,
  manager_id TEXT NOT NULL,
  delegate_id TEXT NOT NULL,
  transaction_scope TEXT NOT NULL,
  start_date TEXT NOT NULL,
  end_date TEXT NOT NULL CHECK (end_date >= start_date),
  acceptance_status TEXT NOT NULL
);

INSERT INTO employees VALUES
  ('E1001', 'Asha Shah', 30),
  ('E1002', 'Dev Rao', 30),
  ('E1003', 'Mina Patel', 30),
  ('E1004', 'Noor Ali', 30),
  ('E1005', 'Ira Das', 30),
  ('E1006', 'Sam Chen', 30),
  ('E1007', 'Lee Park', 30),
  ('E1008', 'Riya Sen', 30);

WITH RECURSIVE dates(d) AS (
  VALUES ('2026-11-01')
  UNION ALL
  SELECT date(d, '+1 day') FROM dates WHERE d < '2026-12-31'
)
INSERT INTO work_calendar
SELECT d, CASE WHEN strftime('%w', d) IN ('0', '6') THEN 0 ELSE 8 END
FROM dates;

INSERT INTO absence_requests VALUES
  ('R1001', 'E1001', '2026-11-16', '2026-11-18', 'APPROVED', 'D2001', 0),
  ('R1002', 'E1002', '2026-11-16', '2026-11-18', 'DENIED', 'D2001', 0),
  ('R1003', 'E1003', '2026-11-16', '2026-11-18', 'NEEDS_REWORK', 'D2001', 0),
  ('R1004', 'E1004', '2026-11-16', '2026-11-18', 'SUBMITTED', 'D2001', 0),
  ('R1005', 'E1005', '2026-11-16', '2026-11-18', 'APPROVED', 'M2002', 0),
  ('R1006', 'E1006', '2026-11-30', '2026-12-02', 'APPROVED', 'M2002', 0),
  ('R1007', 'E1007', '2026-11-16', '2026-11-18', 'SUBMITTED', NULL, 1),
  ('R1008', 'E1008', '2026-11-16', '2026-11-18', 'CANCELLED', 'M2002', 0);

INSERT INTO daily_allocations VALUES
  ('R1001', '2026-11-16', 8, 0),
  ('R1001', '2026-11-17', 8, 0),
  ('R1001', '2026-11-18', 0, 8),
  ('R1005', '2026-11-16', 8, 0),
  ('R1005', '2026-11-17', 8, 0),
  ('R1005', '2026-11-18', 8, 0),
  ('R1006', '2026-11-30', 0, 8),
  ('R1006', '2026-12-01', 8, 0),
  ('R1006', '2026-12-02', 0, 8);

-- Worked time is a separate input, not inferred from approval state.
INSERT INTO worked_time_month VALUES
  ('E1001', '2026-11', 144),
  ('E1002', '2026-11', 168),
  ('E1005', '2026-11', 144);

INSERT INTO delegations VALUES
  ('DG01', 'M2001', 'D2001', 'ABSENCE_APPROVAL', '2026-11-09', '2026-11-20', 'ACCEPTED'),
  ('DG02', 'M2001', 'D2009', 'ABSENCE_APPROVAL', '2026-10-01', '2026-10-31', 'ACCEPTED'),
  ('DG03', 'M2001', 'D2008', 'ABSENCE_APPROVAL', '2026-11-09', '2026-11-20', 'REJECTED');
```

### Report 1 â Approved paid/unpaid allocations

**Business question:** Which approved requests have calculated paid and unpaid units? A left join keeps an approved request without results visible as awaiting processing; its missing values are not mistaken for a completed zero-hour calculation.

```sql
SELECT r.request_id, r.employee_id,
       SUM(d.paid_hours) AS paid_hours,
       SUM(d.unpaid_hours) AS unpaid_hours,
       CASE WHEN COUNT(d.absence_date) = 0
            THEN 'AWAITING_PROCESSING' ELSE 'HAS_ALLOCATION_ROWS' END AS result_state
FROM absence_requests r
LEFT JOIN daily_allocations d ON d.request_id = r.request_id
WHERE r.conceptual_status = 'APPROVED'
GROUP BY r.request_id, r.employee_id
ORDER BY r.request_id;
```

**Expected output:**

| Request ID | Employee ID | Paid hours | Unpaid hours | Result state |
| --- | --- | ---: | ---: | --- |
| `R1001` | `E1001` | 16 | 8 | `HAS_ALLOCATION_ROWS` |
| `R1005` | `E1005` | 24 | 0 | `HAS_ALLOCATION_ROWS` |
| `R1006` | `E1006` | 8 | 16 | `HAS_ALLOCATION_ROWS` |

Submitted, Denied, Needs Rework, and Cancelled requests do not enter this report. Having allocation rows alone does not prove a real payroll run has finalized.

### Report 2 â November hours and illustrative gross pay

**Business question:** How do scheduled hours, verified worked hours, and approved paid absence explain the gross amount? This sample report assumes the illustrated approved events have complete allocation results.

```sql
WITH allocations AS (
  SELECT r.employee_id,
         SUM(d.paid_hours) AS paid_hours,
         SUM(d.unpaid_hours) AS unpaid_hours
  FROM absence_requests r
  JOIN daily_allocations d ON d.request_id = r.request_id
  WHERE r.conceptual_status = 'APPROVED'
    AND substr(d.absence_date, 1, 7) = '2026-11'
  GROUP BY r.employee_id
), schedule AS (
  SELECT SUM(scheduled_hours) AS scheduled_hours
  FROM work_calendar
  WHERE substr(work_date, 1, 7) = '2026-11'
)
SELECT e.employee_id, e.employee_name, s.scheduled_hours,
       w.verified_worked_hours AS worked_hours,
       COALESCE(a.paid_hours, 0) AS paid_absence_hours,
       COALESCE(a.unpaid_hours, 0) AS unpaid_absence_hours,
       w.verified_worked_hours + COALESCE(a.paid_hours, 0)
         + COALESCE(a.unpaid_hours, 0) AS accounted_hours,
       ROUND((w.verified_worked_hours + COALESCE(a.paid_hours, 0))
             * e.hourly_rate, 2) AS gross_pay
FROM employees e
JOIN worked_time_month w ON w.employee_id = e.employee_id
CROSS JOIN schedule s
LEFT JOIN allocations a ON a.employee_id = e.employee_id
WHERE w.pay_month = '2026-11'
ORDER BY e.employee_id;
```

**Expected output, interpreted for the business reader:**

| Employee/test branch | Worked | Paid absence | Unpaid absence | Accounted/scheduled | Gross |
| --- | ---: | ---: | ---: | ---: | ---: |
| Asha â main approved case | 144 h | 16 h | 8 h | 168/168 h | CAD 4,800 |
| Dev â denied request, all scheduled time worked | 168 h | 0 h | 0 h | 168/168 h | CAD 5,040 |
| Ira â fully paid approved comparison | 144 h | 24 h | 0 h | 168/168 h | CAD 5,040 |

The shared schedule fits these synthetic employees only. Real reporting would need each employee's effective schedule, applicable calendar, completed processing results, and authorized payroll rules.

### Report 3 â Outstanding approval ownership

**Business question:** Which submitted requests have an assigned approver, and which need route repair?

```sql
SELECT request_id, employee_id, assigned_approver,
       CASE WHEN routing_exception = 1
            THEN 'ROUTING_EXCEPTION' ELSE 'ASSIGNED' END AS routing_state
FROM absence_requests
WHERE conceptual_status = 'SUBMITTED'
ORDER BY request_id;
```

**Expected output:**

| Request ID | Employee ID | Assigned approver | Routing state |
| --- | --- | --- | --- |
| `R1004` | `E1004` | `D2001` | `ASSIGNED` |
| `R1007` | `E1007` | `NULL` | `ROUTING_EXCEPTION` |

This separates waiting for a decision from waiting for ownership to be repaired.

### Report 4 â Cross-month allocation

**Business question:** Which month contains each calculated absence day?

```sql
SELECT substr(d.absence_date, 1, 7) AS absence_month,
       SUM(d.paid_hours) AS paid_hours,
       SUM(d.unpaid_hours) AS unpaid_hours
FROM absence_requests r
JOIN daily_allocations d ON d.request_id = r.request_id
WHERE r.request_id = 'R1006' AND r.conceptual_status = 'APPROVED'
GROUP BY substr(d.absence_date, 1, 7)
ORDER BY absence_month;
```

**Expected output:**

| Absence month | Paid hours | Unpaid hours |
| --- | ---: | ---: |
| `2026-11` | 0 | 8 |
| `2026-12` | 8 | 8 |

The grouping uses the absence date; a real payroll-period report might use different calendars or pay-period identifiers.

### Report 5 â Delegation candidates for the review date

**Business question:** Which sample delegation passes the transaction/date/acceptance filters on 12 November?

```sql
SELECT delegation_id, manager_id, delegate_id
FROM delegations
WHERE manager_id = 'M2001'
  AND transaction_scope = 'ABSENCE_APPROVAL'
  AND acceptance_status = 'ACCEPTED'
  AND '2026-11-12' BETWEEN start_date AND end_date
ORDER BY delegation_id;
```

**Expected output:**

| Delegation ID | Manager ID | Delegate ID |
| --- | --- | --- |
| `DG01` | `M2001` | `D2001` |

The matching record represents Daniel's delegation to Elena. The expired and rejected examples are excluded. This is a candidate filter, not a security implementation: real identity, eligibility, relationship, revocation, and decision-time authority still require validation.

## Proposed user acceptance testing

These scenarios describe how stakeholders could validate the requirements in an authorized PeopleSoft environment. **PeopleSoft execution status: Not Run.** An expected result in this table is not a claim that the feature has passed UAT.

| ID | Scenario/action | Expected result | Stories |
| --- | --- | --- | --- |
| U01 | Eligible employee selects 16â18 November on the main schedule | 24 working hours | US01, US03 |
| U02 | Employee opens another person's request directly | Access denied without protected data disclosure | US02, US18 |
| U03 | End date is before start date | Clear error; no valid submission | US03 |
| U04 | Required request detail is missing | Correction explained before submission | US03 |
| U05 | Forecast 24 hours with 16 available | Expected 16 paid/8 unpaid, identified as an estimate | US04 |
| U06 | Employee declines the unpaid component | Revise/withdraw path; no submission relying on that split | US04 |
| U07 | Repeat the submission action | One business submission and identifier | US05 |
| U08 | Accepted delegation covers transaction and decision date | Authorized delegate can review within scope | US06 |
| U09 | Delegation is expired, rejected, revoked, or unrelated | No unauthorized decision; validated alternative or exception | US06âUS07 |
| U10 | No eligible approver; administrator repairs route | Exception and authorized reassignment recorded; no auto-approval | US05, US07 |
| U11 | Valid reviewer approves an unchanged acceptable request | Decision communicated; eligible for later processing | US08, US11 |
| U12 | Deny with a reason; employee works all scheduled time | No approved allocation; example gross CAD 5,040 | US09 |
| U13 | Supported first-step Push Back with comment | Needs Rework and clear employee action; no approved allocation | US10âUS11 |
| U14 | Correct and resubmit a returned request | Validation, forecast, and route repeated; history preserved | US10 |
| U15 | Permitted cancellation before processing | Cancelled event excluded from later allocation | US12 |
| U16 | Correction requested after processing/finalization | Controlled authorized correction and reconciliation | US12 |
| U17 | Opening 8 h plus eligible November credit 8 h | 16 h available; no duplicate credit | US13 |
| U18 | Process Asha's approved event | 16 paid/8 unpaid; closing balance zero | US14 |
| U19 | Reconcile main November hours at CAD 30 | 160 payable h; CAD 4,800 gross | US15 |
| U20 | Another approved event reduces the balance after acknowledgement | Changed split shown; renewed agreement before approval relies on it | US04, US08 |
| U21 | Date range includes off-days or supported partial days | Duration follows actual schedule and configured units | US03 |
| U22 | Delegation expires while a request is awaiting decision | Decision-time authority fails; validated reroute/exception | US06âUS07 |
| U23 | Replay decision or restart processing | No duplicate entitlement consumption or paid units | US08, US14 |
| U24 | Allocate 30 Novemberâ2 December with the dated credit | Daily 0/8, 8/0, 0/8 paid/unpaid h | US13âUS14 |
| U25 | Report mixed statuses and attempt an unauthorized export | Correct populations and enforced report/data access | US16, US18 |
| U26 | Later-stage reviewer uses supported Push Back | Configured previous-reviewer route, not assumed direct employee return | US10, US17 |

I would also include keyboard/accessibility review, audit-history checks, and a message walkthrough for NFR02âNFR05. A real execution record would capture environment/release, test data, actual result, evidence, executor, date, and any defect and retest reference.

### What was checked in this exercise

The embedded SQL was executed against an in-memory SQLite database. Checks covered the twenty-one-weekday November schedule, the 16/8 main allocation, the CAD 4,800 gross result, the denied and fully paid comparison results, cross-month grouping, outstanding approval ownership, and the accepted delegation candidate. The calculation tables were also checked against the dated-credit assumptions.

The sample database key prevents duplicate allocation rows for the same request/date. It does not establish complete workflow idempotency, application security, or concurrent transaction handling. PeopleSoft configuration, workflow execution, notification delivery, security, accessibility, and UAT remain outside the executed evidence.

## Business analysis techniques and academic connections

This PeopleSoft case applies requirements analysis, systems modelling, database reasoning, and technical writing associated with the Honours Bachelor of Technology â Software Development programme at Seneca. The academic connections support the analysis of the absence process.

The connections below use related subjects in Seneca's published BSD curriculum. They identify transferable skills; they do not suggest that those courses taught PeopleSoft or replace an academic transcript. Course codes and outlines can differ by academic year. [S01âS05]

| Related course or area | Connection demonstrated in this case |
| --- | --- |
| BTS330 â Business Requirements Analysis Using OO Models [S03] | Problem definition, stakeholder questions, business rules, and written requirements |
| BTS435 â systems analysis and design [S02] | User stories, acceptance criteria, process decisions, and backlog refinement |
| BTD210 â Database Design Principles [S04] | Data entities, keys, relationships, SQL joins, and report definitions |
| BTD315 â advanced database study [S02] | Aggregation, date-based reporting, and checking data integrity |
| BTS535 â software project management [S02] | Scope, priorities, risks, dependencies, and a definition of completion |
| BTC440 â Business and Technical Writing [S05] | A readable case narrative, clear requirements, and source attribution |
| ENG106 â writing/research; BTC240 â organizational communication [S02] | Separating evidence from assumptions and adapting explanations to stakeholders |
| BTH545 â user-centred interface design [S02] | Employee-facing validation, understandable status messages, and accessibility questions |
| BTP405 â development/design methods [S02] | Testable behavior, quality checks, and separating analysis from implementation |
| BAB100 â business foundations [S02] | Understanding how operations, HR, management, and payroll share a process |

The case demonstrates requirements analysis, process modelling, user-story writing, acceptance-criteria design, SQL reporting, numerical reconciliation, documentation research, and communication. My PeopleSoft knowledge here is at an introductory functional level: terminology, request flow, forecasting, approval, delegation, security concepts, and the distinction between approval and processing.

## Coverage of the thirteen learning topics

Some topics explain the system context rather than appearing as employee actions. I kept that distinction so the process diagram follows a believable request journey.

| Learning topic | Where it appears in this case | Oracle references |
| --- | --- | --- |
| 1. Learning resources | Research approach and reference list | O01 |
| 2. PeopleSoft suites | HCM scope; FSCM/Campus context | O02 |
| 3. Browser access and PeopleTools | Employee access and functional configuration dependencies | O03, O03A |
| 4. Fluid homepages and tiles | Employee self-service and review navigation | O04, O07, O08A |
| 5. Permissions and data scope | US02/US18, access matrix, and negative tests | O05, O05A |
| 6. Absence terminology | Vacation name/reason, entitlement, take, and glossary | O06âO06C |
| 7. Employee request process | US01âUS05 and Asha's submission | O07 |
| 8. Manager review | Approve/Deny/Push Back branches and US08âUS11 | O08, O08A |
| 9. Configuration and business rules | Assumptions, requirements, and fit-gap questions | O09 |
| 10. Forecasting | Main estimate, changed balance, and dated-credit examples | O10 |
| 11. Post-approval processing | Daily allocation, monthly reconciliation, and US13âUS15 | O11âO12 |
| 12. Query and reporting | Reporting questions and independent SQL exercise | O14 |
| 13. Delegation | Daniel/Elena, routing exceptions, and US06âUS07 | O13 |

## Presenting the backlog in GitHub

This README is the complete case-study document. The same user stories can also become individual GitHub Issues; that does not require additional repository files or an application. [G04]

1. Create a repository, for example `peoplesoft-absence-ba-case-study`, and upload this file as `README.md` in its root. Upload `swimlane-diagram.png` alongside it so the image link renders. [G01âG02]
2. In the repository, choose **Issues â New issue**. Use one issue per story, starting with `US01 â Request time for a family event`.
3. Copy the matching story statement, acceptance criteria, and trace line into the issue body. Use checklist boxes for the criteria if helpful.
4. Add labels such as `user-story`, `analysis`, and `must` or `should`. An optional `needs-validation` label identifies behavior that still needs an environment or policy decision.
5. Repeat for US02âUS18. Link related story IDs or issue numbers so the sequence and dependencies remain visible.

For example, US08 depends on the submission in US05, valid review authority in US06/US07, and the current split in US04. US14 depends on an eligible approval, and US15 depends on calculated allocations plus verified worked time.

An issue body can use this structure:

```markdown
## User story

As an employee, I want to request vacation for my sister's wedding
so that I can arrange the time away and receive a clear decision.

## Acceptance criteria

- [ ] Given the eight-hour weekday schedule, selecting 16â18 November
      produces twenty-four requested working hours.
- [ ] Before proceeding, the employee can understand whether all
      requested hours are expected to be paid.

## Traceability

US01 Â· FR01 Â· BR01âBR02 Â· U01 Â· Must

## Evidence status

Documented learning scenario; PeopleSoft implementation and UAT not performed.
```

For this documentation project, **analysis complete** means that the story has a clear user need, reviewable criteria, traced requirements/tests, and a consistent place in the process. It does not mean that its behavior has been built or passed in PeopleSoft. If using a board, stages such as *Draft*, *Ready for analysis review*, and *Analysis documented* make that boundary visible.

## What I learned and what I would explore next

The main lesson from this exercise is that a leave request has more than one outcome to explain. Approval answers whether the time away is authorized. Entitlement rules answer how much may be paid. Processing and verified worked time support the eventual pay calculation. Treating those as separate questions made the requirements clearer.

I also found that exceptions deserve their own requirements. An unavailable manager, an expired delegation, a changed balance, or a processed cancellation can interrupt an otherwise simple flow. Identifying those cases helped me move from describing screens to describing responsibilities and decisions.

My next learning step would be to validate the fit-gap questions in a permitted training environment, compare the proposed states with delivered labels, and execute the UAT scenarios with documented evidence. Until then, the result is a researched BA case study with checked synthetic calculations.

## References

The Oracle sources informed the product concepts and workflow interpretation. The Seneca sources support the academic skill connections. The fictional company policy, story, and numerical examples were developed for this exercise; Oracle does not prescribe those numbers. Sources reviewed for this study: **8 October 2026**. Application documentation below uses HCM 9.2.055 and PeopleTools 8.63 where specified; a real organization's release and configuration must be checked.

### Oracle documentation and learning resources

| ID | Source | Used for |
| --- | --- | --- |
| O01 | [PeopleSoft Training Center](https://docs.oracle.com/cd/E52319_01/infoportal/training.html) | Learning-resource context |
| O02 | [PeopleSoft documentation](https://docs.oracle.com/en/applications/peoplesoft/) | Product-suite scope |
| O03 | [Understanding PeopleSoft Browser-Based Applications](https://docs.oracle.com/en/applications/peoplesoft/peopletools/8.63/applications-user-s-guide/understanding-peoplesoft-browser-based-applications.html) | Browser access and navigation |
| O03A | [PeopleTools Development Environment](https://docs.oracle.com/en/applications/peoplesoft/peopletools/8.63/getting-started-with-peopletools/peopletools-development-environment.html) | Technical/functional tool context |
| O04 | [Understanding Fluid Homepages](https://docs.oracle.com/en/applications/peoplesoft/peopletools/8.63/applications-user-s-guide/understanding-fluid-homepages.html) | Homepages and tiles |
| O05 | [Understanding Permission Lists](https://docs.oracle.com/en/applications/peoplesoft/peopletools/8.63/security-administration/understanding-permission-lists.html) | Roles, user profiles, and function access |
| O05A | [Understanding Search Records](https://docs.oracle.com/en/applications/peoplesoft/peopletools/8.63/application-designer-developer-s-guide/understanding-search-records.html) | Record-access considerations |
| O06 | [Defining Absence Types and Reasons](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-absence-management/defining-absence-types-reasons.html) | Absence categories and reasons |
| O06B | [Absence Takes](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-absence-management/absence-takes.html) | Take rules and entitlement use |
| O06C | [PeopleSoft Absence Management reference PDF](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-absence-management/peoplesoft-absence-management.pdf) | Entitlement and broader functional terminology |
| O07 | [Using PeopleSoft Fluid User Interface to Work with Absences as an Employee](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-absence-management/using-peoplesoft-fluid-user-interface-work-absences-employee.html) | Employee request, balance, and status tasks |
| O08 | [Approval Framework and Absence Management](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-absence-management/approval-framework-absence-management.html) | Approval, denial, pushback, and routing exceptions |
| O08A | [Setting Up and Using Fluid Approvals](https://docs.oracle.com/en/applications/peoplesoft/peoplesoft-common/enterprise-components/setting-using-fluid-approvals.html) | Approval interface, supported actions, and administration |
| O09 | [Defining Self-Service Absence Rules for a Take Element](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-absence-management/defining-self-service-absence-rules-take-element.html) | Units, request validation, approval, and cancellation setup |
| O10 | [Setting Up Absence Entitlement Balance Forecasting and Inquiry](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-absence-management/setting-absence-entitlement-balance-forecasting-inquiry.html) | Forecasting calendars, filters, and estimates |
| O11 | [Take Features](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-global-payroll/take-features.html) | Configurable treatment of insufficient entitlement |
| O12 | [Absence Entitlement and Absence Take Processes](https://docs.oracle.com/en/applications/peoplesoft/human-capital-management/9.2.055/peoplesoft-global-payroll/absence-entitlement-absence-take-processes.html) | Entitlement calculation, absence processing, and payroll relationship |
| O13 | [Working with Self-Service Delegation](https://docs.oracle.com/en/applications/peoplesoft/peoplesoft-common/enterprise-components/working-self-service-delegation.html) | Transaction scope, dates, delegates, and acceptance |
| O14 | [PeopleSoft Query Overview](https://docs.oracle.com/en/applications/peoplesoft/peopletools/8.63/query/peoplesoft-query-overview.html) | Reporting concepts and Query context |

### Seneca academic references

| ID | Source | Used for |
| --- | --- | --- |
| S01 | [Honours Bachelor of Technology â Software Development](https://www.senecapolytechnic.ca/programs/fulltime/BSD.html) | Programme context and transferable skills |
| S02 | [BSD published course curriculum](https://www.senecapolytechnic.ca/programs/fulltime/BSD/courses.html) | Related analysis, design, database, project, and communication subjects |
| S03 | [BTS330 course outline](https://www.senecapolytechnic.ca/cgi-bin/subject?s1=BTS330) | Requirements research, analysis, and documentation |
| S04 | [BTD210 course outline](https://apps.senecapolytechnic.ca/ssos/findOutline.do?subjectCode=BTD210) | Relational design, schemas, and SQL |
| S05 | [BTC440 course outline](https://www.senecapolytechnic.ca/cgi-bin/subject?s1=BTC440) | Business/technical writing and research attribution |

### GitHub documentation

| ID | Source | Used for |
| --- | --- | --- |
| G01 | [Creating a new repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) | Repository setup |
| G02 | [Adding a file to a repository](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository) | Publishing the README |
| G04 | [Creating an issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/creating-an-issue) | One GitHub Issue per user story |
