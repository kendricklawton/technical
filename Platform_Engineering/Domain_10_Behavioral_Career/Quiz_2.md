# PE Domain 10: Behavioral & Career — Scenario Quiz 2

Test your understanding of conflict resolution, saying no, and cross-team collaboration.

---

### Question 1
A Product Engineering Director approaches your platform team and demands: "Drop everything on your roadmap and build a custom deployment pipeline for my team's unique framework in 2 weeks." How should a Senior Platform Engineer handle this situation?

A) Immediately agree and cancel all planned roadmap items to avoid conflict.  
B) Listen with empathy to understand the underlying business driver and timeline; evaluate the request against current organizational priorities using Cost of Delay / RICE; propose an escape hatch or existing Golden Path alternative; and involve your Product Manager/EM to align on trade-offs.  
C) Refuse rudely and tell the Director to read the company documentation.  
D) Complain publicly in the company Slack `#general` channel.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Senior engineers handle stakeholder pressure with commercial empathy and structured prioritization. They understand the "why" behind the request, evaluate organizational trade-offs transparently, and seek collaborative alternatives rather than reacting emotionally.
</details>

---

### Question 2
Two senior engineers on your platform team are locked in a heated ideological deadlock: one insists on migrating to Crossplane for all cloud provisioning, while the other insists on staying with Terraform and Atlantis. The dispute is stalling team momentum. How do you resolve this?

A) Let them argue until one engineer resigns.  
B) Author a structured Architecture Decision Record (ADR) defining explicit evaluation criteria (developer experience, operational complexity, ecosystem maturity, team skill set), build a time-boxed 2-week spike on a shared real-world use case, test both against the criteria with real data, and facilitate a consensus review.  
C) Mandate that both tools be completely banned.  
D) Flip a coin to decide.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Resolving architectural deadlocks requires moving from subjective opinions to objective, empirical data. A structured spike evaluated against agreed-upon criteria depersonalizes the debate and drives evidence-based technical consensus.
</details>

---

### Question 3
What is the primary purpose of an **Architecture Decision Record (ADR)** in an engineering organization?

A) To grade developer performance.  
B) To replace source code documentation.  
C) To document significant architectural decisions, context, options considered, trade-offs, and consequences in a lightweight, version-controlled markdown file, preserving institutional memory and explaining *why* decisions were made to future engineers.  
D) To file patents with the government.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** ADRs capture the historical context of architectural choices. When new engineers join 2 years later, ADRs prevent endless re-litigation of settled decisions by clearly articulating the trade-offs and constraints that existed at the time.
</details>

---

### Question 4
In Will Larson's *Staff Engineer*, which archetype describes a Staff engineer who embeds with leadership, sets broad organizational technical strategy, and coordinates cross-departmental roadmaps?

A) The Solver  
B) The Tech Lead  
C) The Architect  
D) The Right Hand  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Larson identifies 4 primary Staff archetypes: Tech Lead, Architect, Solver, and Right Hand. The Architect archetype focuses on broad technical strategy, system design consistency, and aligning architectural roadmaps across entire organizations.
</details>

---

### Question 5
When an interviewer asks: "Tell me about a time you disagreed with your manager or team lead", what is the primary behavior they are evaluating?

A) "Have Backbone; Disagree and Commit": whether you can respectfully challenge decisions with data and logical arguments, explore alternatives constructively, and once a final decision is made, commit fully to the team's direction without passive-aggressive sabotage.  
B) Refusing to work until you get your way.  
C) Whether you talk behind people's backs.  
D) Blind obedience without questioning authority.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Healthy organizations seek leaders who courageously present data-backed counterarguments to prevent bad decisions, but who possess the professional maturity to execute the final decision wholeheartedly once the debate concludes.
</details>

---

### Question 6
What is the concept of **"Glue Work"** in engineering teams, as coined by Tanya Reilly?

A) Running database backups.  
B) Physically repairing broken office chairs.  
C) Writing code that glues two APIs together.  
D) Essential non-technical or organizational work that holds projects together (onboarding new hires, improving documentation, coordinating cross-team meetings, unblocking teammates, triaging tickets), which is critical for team success but often under-rewarded in traditional promo cycles if not quantified.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Glue work is vital for organizational health. Senior engineers must be able to perform glue work while articulating its strategic business impact (e.g., "Reduced onboarding ramp time by 50% for 30 engineers").
</details>

---

### Question 7
How should you frame quantifiable metrics in your behavioral STAR stories?

A) Avoid numbers entirely because they sound fabricated.  
B) Only mention lines of code written.  
C) Say "it made things much faster".  
D) Anchor outcomes with baseline and final metrics: "Reduced deployment lead time from 14 days to 45 minutes across 35 teams, saving an estimated 1,200 developer hours annually and lowering change failure rate from 28% to 4%."  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Quantified results demonstrate tangible business impact. Stating the baseline, final outcome, and scale (teams, hours, percentages) proves you understand the commercial return on platform engineering investments.
</details>

---

### Question 8
What is the best way to handle the reverse-interview portion ("Do you have any questions for me?") at the end of a behavioral round?

A) Ask only about vacation days and free lunches.  
B) Ask what time the interview ends.  
   - "Can a developer go from an empty repo to production without filing a ticket or messaging someone?"
   - "What fraction of the platform team's time goes to manual unplanned requests versus roadmap features?"
   - "How does the organization balance feature velocity against technical debt and reliability?"  
C) Ask insightful, calibrated questions that probe the company's real engineering culture, platform maturity, and challenges:  
D) "No, I think you covered everything."  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Reverse-interviewing is a powerful leveling signal. Senior engineers ask sharp, diagnostic questions that reveal operational reality, showing they understand the common failure modes of internal platforms.
</details>

---

### Question 9
During an interview, you realize that an architecture you proposed has a major flaw that the interviewer just pointed out. What is the most mature response?

A) Acknowledge the flaw immediately with intellectual honesty: "That's an excellent catch. Under high network partition, my design would indeed cause split-brain state. To mitigate that, I would introduce Raft consensus leases or fencing tokens..." and iterate on the design collaboratively.  
B) Blame the whiteboard marker.  
C) Give up and remain silent for the rest of the interview.  
D) Defend the flawed design aggressively and deny the problem exists.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Interviewers deliberately probe for weaknesses to test how candidates handle feedback. Being defensive is an immediate fail. Embracing the feedback, validating the edge case, and collaboratively pivoting demonstrates senior humility and problem-solving agility.
</details>

---

### Question 10
In Camille Fournier's *The Manager's Path*, what is the core expectation of a Senior Individual Contributor?

A) Being self-directed, reliable, capable of taking ambiguous complex projects from concept to completion, mentoring junior engineers, and proactively identifying operational risks before they cause outages.  
B) Writing all company blog posts.  
C) Writing code 100% of the day without talking to others.  
D) Managing payroll and firing employees.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Senior engineers require minimal supervision. They own projects end to end, uplift their peers through mentorship and code reviews, and maintain a proactive stance toward platform reliability.
</details>
