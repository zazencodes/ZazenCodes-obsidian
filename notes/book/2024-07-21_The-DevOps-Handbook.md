---
date: 2024-07-21
tags:
    - book
hubs:
    - "[[devops]]"
---

# The DevOps Handbook
Author: Gene Kim et al.

# Curated Notes

The first section has curated notes based on my highlights. The highlight section below this has my favorite passages from the book.

# Index
- Software Development Practices
- Continuous Delivery & Deployment
- Development & Testing
- Feature Management
- Software Architecture
- Monitoring & Telemetry


# Software Development Practices

## Making Work Visible
- **Kanban Boards**:
  - Represent work as cards.
  - Help manage work flow from left to right quickly.
  - Should span the entire value stream from business hypothesis to “done”.



## Major Causes of Wasted Effort in Software Development
- **Partially Done Work**: Becomes obsolete and loses value over time.
- **Extra Processes**: Additional work that doesn’t add value to the customer.
- **Extra Features**: Unnecessary features for the organization or customer.
- **Task Switching**: Effort required to context switch and manage dependencies between projects.



## Feedback Loops
- Enable quick detection and recovery of problems.
- Inform prevention of future problems.
- Increase quality, safety, and create organizational learning.



## Andon Cord Practice
- **Process**:
  - Pull the cord, alert team leader to resolve the problem.
  - If unresolved within specified time, halt production.
- **Benefits**:
  - Prevents loss of critical information.
  - Enables learning in complex systems.



## Problem-Solving Approach
- Core of the Toyota Production System.
- Leaders coach with questions:
  - What was your last step and what happened?
  - What did you learn?
  - What is your condition now?
  - What is your next target condition?
  - What obstacle are you working on now?
  - What is your next step?
  - What is your expected outcome?
  - When can we check?



## Demonstrating Early Wins
- Break up larger improvement goals into small, incremental steps.
- Detect errors early and correct quickly.



# Continuous Delivery & Deployment
## Continuous Delivery
- Create automated deployment pipeline foundations.
- Ensure automated tests validate deployable state.
- Daily code integration by developers.
- Architect environments for low-risk releases.



## Fast Flow of Work
- Ensure production-like environments are available on demand.
- Document all assets required to create the application environment in version control.



## Repeatable Environment Creation Systems
- Increase capacity by adding more servers.
- Ensure all changes are replicated automatically.
- Put changes into version control.



## Developer Responsibilities
- Write, test, and run code in a production-like environment.
- Deployment pipeline ensures code is built and tested in production-like environment.
- Detect and fix build, test, or integration errors immediately.



## Automated Performance Tests
- Validate performance across entire application stack.
- Detect problems early when fixes are cheapest and fastest.



## Trunk-Based Development Practice
- Developers check in code to trunk daily.
- Reduces batch size and facilitates single-piece flow.
- Frequent code commits ensure automated tests and quick error detection.
- Trunk-based development predicts higher throughput, stability, job satisfaction, and lower burnout.



## Case Study: Etsy-Self-Service Developer Deployment (2014)
- Engineers deploy code 25-50 times a day.
- Use chat room to manage deployment queue.
- Run 4,500 unit tests in under one minute before checking in code.
- Continuous integration servers run over 7,000 automated tests.
- Use internal tool Deployinator for deployment.
- Deployments visible to all engineers.



# Development & Testing
## Deployments vs. Releases
- **Deployment**: Installation of a specified version of software to an environment.
- **Release**: Making features available to all or a segment of customers.



## Deployment Patterns
- **Blue-Green Deployment**:
  - Test new version in inactive environment.
  - Direct traffic to blue environment once confident.
- **Canary Release**:
  - Promote code to larger environments as confidence grows.
  - Detects issues early like canaries in coal mines.



## Database Handling
- **Two Database Approach**:
  - Blue (old) and green (new) databases.
  - Read-only mode for blue during release.
- **Decoupling Database Changes**:
  - Only additive changes.
  - No assumptions about database version in production.



# Feature Management
## Feature Toggles
- Selectively enable/disable features without production code deployment.
- Control feature visibility for specific user segments.



## Case Study: Dark Launch of Facebook Chat (2008)
- Chat initially visible only to internal employees.
- Simulate production-like loads during development.
- Incremental release to larger customer segments.



## Creating Variations
- Focus on outcomes, not the form.
- Deployments should be low-risk, push-button events on demand.



# Software Architecture
## Service-Oriented Architecture
- Allows small teams to deploy independently, quickly, and safely.



## Monolithic Architectures
- Often best choice early in product life cycle.



## Strangler Application Pattern
- Place existing functionality behind an API.
- Implement new functionality with desired architecture.



# Monitoring & Telemetry
## Telemetry and Metrics
- Centralize logs and transform into metrics.
- Perform statistical operations for early problem detection.
- Collect telemetry from deployment pipeline.




## Instrumentation of Features
- Every feature should generate production telemetry.
- Confirm operation as designed and desired outcomes.



## Production Telemetry
- Reveals problems in design and operations.



## Leading Indicators
- Metrics to warn of deviations from standard operations:
  - Application level: increasing web page load times.
  - OS level: low server free memory.
  - Database level: longer transaction times.
  - Network level: fewer functioning servers behind load balancer.
- Configure alerting systems for early corrective action.



## Non-Gaussian Distribution in Production Data
- Many data sets have chi squared distribution.



## Automation in Chat Rooms
- Benefits:
  - Transparent daily work.
  - Encourages asking for help.
  - Enables rapid organizational learning.
  - Public communication vs. private emails.


# Highlights



The purpose of the DevOps Handbook is to give you the theory, principles, and practices you need to successfully start your DevOps initiative and achieve your desired outcomes.


---



To help us see where work is flowing well and where work is queued or stalled, we need to make our work as visible as possible. Kanban boards represent work as cards and helps us manage our work so that it flows from left to right as quickly as possible. Ideally, our kanban board will span the entire value stream, from business hypothesis to “done” when it reaches the right side of the board.

---


Major causes of wasted effort in software development:
- Partially done work: Partially done work becomes obsolete and loses value as time progresses.
- Extra processes: Any additional work that is being performed in a process that does not add value to the customer.
- Extra features: Features built into the service that are not needed by the organization or the customer.
- Task switching: Requires effort to context switch and manage dependencies between projects.

---


Feedback loops not only enable quick detection and recovery of problems, they also inform us on how to prevent these problems from occurring again in the future. Doing this increases the quality and safety of our system of work, and creates organizational learning.

---



When the Andon cord is pulled, the team leader is alerted and immediately works to resolve the problem. If the problem cannot be resolved within a specified time (e.g, fifty-five seconds), the production line is halted so that the entire organization can be mobilized to assist with problem resolution until a successful countermeasure has been developed.
Instead of working around the problem or scheduling a fix "when we have more time," we swarm to fix it immediately
This practice of swarming seems contrary to common management practice, as we are deliberately allowing a local problem to disrupt operations globally.
However, swarming enables learning. It prevents the loss of critical information due to fading memories or changing circumstances. This is especially critical in complex systems, where many problems occur because of some unexpected, idiosyncratic interaction of people, processes, products, places, and circumstances-as time passes, it becomes impossible to reconstruct exactly what was going on when the problem occurred.



---


The leader helps coach the person conducting the experiment with questions that may include:
- What was your last step and what happened?
- What did you learn?
- What is your condition now?
- What is your next target condition?
- What obstacle are you working on now?
- What is your next step?
- What is your expected outcome?
- When can we check?
This problem-solving approach in which leaders help workers see and solve problems in their daily work is at the core of the Toyota Production System.

---


Regardless of how we scope our initial effort, we must demonstrate early wins and broadcast our successes. We do this by breaking up our larger improvement goals into small, incremental steps. This not only creates our improvements faster, it also enables us to discover when we have made the wrong choice of value stream-by detecting our errors early, we can quickly back up and try again, making different decisions armed with our new learnings.

---


Continuous delivery includes creating the foundations of our automated deployment pipeline, ensuring that we have automated tests that constantly validate that we are in a deployable state, having developers integrate their code in to trunk daily, and architecting our environments and code to enable low-risk releases.


---


The fast flow of work from Development to Operations requires that anyone can get production-like environments on demand.

---


Document all assets required to create the application environment in version control, including
- All application code and dependencies e.g., libraries, static content, etc.)
- Any script used to create database schemas, application reference data, etc.
- All the environment creation tools and artifacts described in the previous step (e.g, VMware or AMI images, Puppet or Chef recipes, etc.)
- Any file used to create containers (e.g., Docker or Rocket definition or composition files)
- All supporting automated tests and any manual test scripts
- Any script that supports code packaging, deployment, database migration, and environment provisioning
- All project artifacts (e.g., requirements documentation, deployment procedures, release notes, etc.)
- All cloud configuration files (e.g., AWS Cloudformation templates,
-icrosoft Azure Stack DSC files, OpenStack HEAT)
- Any other script or configuration information required to create infrastructure that supports multiple services le.., enterprise service buses, database management systems, DNS zone files, configuration rules for firewalls, and other networking devices).


---



By having repeatable environment creation systems, we are able to easily increase capacity by adding more servers into rotation (i.e., horizontal scaling).

Instead of manually logging into servers and making changes, we must make changes in a way that ensures all changes are replicated everywhere automatically and that all our changes are put into version control.


---



By having developers write, test, and run their own code in a production-like environment, the majority of the work to successfully integrate our code and environments happens during our daily work, instead of at the end of the release.


---


The deployment pipeline should ensure that all code checked in to version control is automatically built and tested in a production-like environment. By doing this, we find any build, test, or integration errors as soon as a change is intro-duced, enabling us to fix them immediately. Done correctly, this allows us to always be assured that we are in a deployable and shippable state.


---


Our goal is to write and run automated performance tests that validate our performance across the entire application stack (code, database, storage, network, virtualization, etc.) as part of the deployment pipeline, so we detect problems early, when the fixes are cheapest and fastest.


---


Trunk-based development practice requires all developers check in their code to trunk at least once per day. Checking code in this frequently reduces our batch size to the work performed by our entire developer team in a single day. The more frequently developers check in their code to trunk, the smaller the batch size and the closer we are to the theoretical ideal of single-piece flow. Frequent code commits to trunk means we can run all automated tests on our software system as a whole and receive alerts when a change breaks some other part of the application or interferes with the work of another developer.
And because we can detect merge problems when they are small, we can correct them faster.

Trunk-based development is likely the most controversial practice discussed in the book. Many engineers will not believe that it's possible, even those that prefer working uninterrupted on a private branch without having to deal with other developers. However, the data from Puppet Labs' 2015 State of DevOps Report is clear: trunk-based development predicts higher throughput and better stability, and even higher job satisfaction and lower rates of burnout.


---


Case Study
Etsy-Self-Service Developer Deployment, an Example of Continuous Deployment (2014)
Unlike at Facebook where deployments are managed by release engineers, at Etsy deployments are performed by anyone who wants to perform a deployment. It happens 25-50 times a day (as of 2011).
Engineers who want to deploy their code first go to a chat room, where engineers add themselves to the deploy queue, see the deployment activity in progress, see who else is in the queue, broadcast their activities, and get help from other engineers when they need it.
Before the developer even checks in code, they will run on their workstation all 4,500 unit tests, which takes less than one minute. All calls to external systems, such as databases, have been stubbed out.
After they check their changes in to trunk in version control, over seven thousand automated trunk tests are instantly run on their continuous integration (CI) servers.
“The 7,000 trunk tests would take about half an hour to execute. So we split these tests up into subsets, and distribute those onto the 10 machines in our Jenkins [CI] cluster....giving us the desired 11 minute runtime."
The next tests to run are the smoke tests, which are system level tests that run cURL to execute PHPUnit test cases. Following these tests, the functional tests are run, which execute end-to-end GUl-driven tests on a live server.
Once it is an engineer's turn to deploy "you go to Deployinator (an internal tool) and push the button to get it on QA. From there it visits Princess (staging environment). Then, when it's ready to go live, you hit the "Prod" button and soon your code is live, and everyone in chat knows who pushed what code, complete with a link to the diff. For anyone not on chat, there's the email that everyone gets with the same information.


---




Deployments and releases are two distinct actions that serve two very different purposes:
- Deployment is the installation of a specified version of software to a given environment
- Release is when we make a feature (or set of features) available to all our customers or a segment of customers.


---




The simplest deployment pattern is through Blue-Green pattern. To release a new version of our service, we deploy to the inactive environment where we can perform our testing without interrupting the user experience.
When we are confident that everything is functioning as designed, we execute our release by directing traffic to the blue environment. Thus, blue becomes live and green becomes staging.

The canary release pattern automates the release process of promoting to successively larger and more critical environments as we confirm that the code is operating as designed.
The term canary release comes from the tradition of coal miners bringing caged canaries into mines to provide early detection of toxic levels of carbon monoxide. If there was too much gas in the cave, it would kill the canaries before it killed the miners, alerting them to evacuate.


---



Having two versions of our application in production creates problems when they depend upon a common database-when the deployment requires database schema changes or adding, modifying, or deleting tables or columns, the database cannot support both versions of our application. There are two general approaches to solving this problem:
1. Create two databases i.e., a blue and green database): Each version-blue (old) and green (new)—of the application has its own database. During the release, we put the blue database into read-only mode, perform a backup of it, restore onto the green database, and finally switch traffic to the green environment. The problem with this pattern is that if we need to roll back to the blue version, we can potentially lose transactions if we don't manually migrate them from the green version first.
2. Decouple database changes from application changes: Instead of supporting two databases, we decouple the release of database changes from the release of application changes by doing two things: First, we make only additive changes to our database, we never mutate existing database objects, and second, we make no assumptions in our application about which database version will be in production. This is very different than how we've been traditionally trained to think about databases, where we avoid duplicating data.

---


The primary way we enable application-based release patterns is by implementing feature toggles, which provide us with the mechanism to selectively enable and disable features without requiring a production code deployment. Feature toggles can also control which features are visible and available to specific user segments (e.g., internal employees, segments of customers).

---

Case Study Dark launch of Facebook chat (2008)

At first it was made visible to all internal employees, but it was completely hidden from external Facebook users through Gatekeeper, the Facebook feature toggling service.
As part of their dark launch process, every Facebook user session, which runs JavaScript in the user browser, had a test harness loaded into it - the chat Ul elements were hidden, but the browser client would send invisible test chat messages to the back-end chat service that was already in production, enabling them to simulate production-like loads throughout the entire project, allowing them to find and fix performance problems long before the customer release.
By doing this, every Facebook user was part of a massive load testing program, which enabled the team to gain confidence that their systems could handle realistic production-like loads. During the release, they incrementally enabled the chat functionality to ever-larger segments of the customer population -first to all internal Facebook employees, then to 1% of the customer population, then to 5%, and so forth.


---

Every organization should create their variations, based on what they need. The key thing we should care about is not the form, but the outcomes: deployments should be low-risk, push-button events we can perform on demand.


---



For a service like Gmail, there's five or six other layers of services underneath it, each very focused on a very specific function. This kind of service-oriented architecture allows small teams to work on smaller and simpler units of development that each team can deploy inde-pendently, quickly, and safely.


---


Monolithic architectures are not inherently bad—in fact, they are often the best choice for an organization early in a product life cycle.


---




The term strangler application was inspired by strangler vines that seed in the upper branches of fig trees and gradually work their way down the tree until they root in the soil. Over many years they grow into fantastic and beautiful shapes, meanwhile strangling and killing the tree that was their host.
If we have determined that our current architecture is too tightly coupled, we can start safely decoupling parts of the functionality from our existing architecture. The strangler application pattern involves placing existing functionality behind an API, where it remains unchanged, and implementing new functionality using our desired architecture, making calls to the old system when necessary.



---


Ideally, we will create telemetry that tells us exactly when anything of interest happens, as well as where and how.

Once we have centralized our logs, we can transform them into metrics by counting them in the event router, so that we can perform statistical operations on them, such as using anomaly detection to find outliers and variances even earlier in the problem cycle. For instance, we might configure our alerting to notify us if we went from "ten segfaults last week" to "thousands of segfaults in the last hour”

In addition to collecting telemetry from our production services and environments, we must also collect telemetry from our deployment pipeline when important events occur, such as when our automated tests pass or fail and when we perform deployments to any environment. We should also collect telemetry on how long it takes us to execute our builds and tests. By doing this, we can detect conditions that could indicate problems, such as if the performance test or our build takes twice as long as normal, allowing us to find and fix errors before they go into production.


---


In the applications we create and operate, every feature should be instrumented- if it was important enough for an engineer to implement, it is certainly important enough to generate enough production telemetry so that we can confirm that it is operating as designed and that the desired outcomes are being achieved.



---


By generating production telemetry as part of our daily work, we create an ever-improving capability to not only see problems as they occur, but also to design our work so that problems in design and operations can be revealed.



---





if we had an issue where our NGINX web server stopped responding to requests, we would look at the leading indicators that could have warned us earlier that we were starting to deviate from standard operations, such as:
- Application level: increasing web page load times, etc.
- OS level: server free memory running low, disk space running low, etc.
- Database level: database transaction times taking longer than normal, etc.
- Network level: number of functioning servers behind the load balancer dropping, etc.
Each of these metrics is a potential precursor to a production incident. For each, we would configure our alerting systems to notify them when they deviate sufficiently from the mean, so that we can take corrective action.



---


Many production data sets are non-Gaussian distribution. In Operations, many of our data sets have what we call 'chi squared' distribution.


---


Having deployments performed by automation in the chat room (as opposed to running automated scripts via command line) had numerous benefits, including:
- Everyone saw everything that was happening.
- Engineers on their first day of work could see what daily work looked like and how it was performed.
- People were more apt to ask for help when they saw others helping each other.
- Rapid organizational learning was enabled and accumulated.
Furthermore, beyond the above tested benefits, chat rooms inherently record and make all communications public; in contrast, emails are private by default, and the information in them cannot easily be discovered or propagated within an organization.



---





