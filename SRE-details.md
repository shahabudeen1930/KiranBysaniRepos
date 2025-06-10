#### SRE & DevOps
- Explinantion
     - SRE, on the other hand, is a specific implementation of DevOps principles, developed at Google, with a strong focus on engineering reliability through software. It introduces measurable practices like SLIs, SLOs, and error budgets, and it treats operations problems as software problems.
     - DevOps is more of a cultural philosophy that promotes collaboration between development and operations teams, emphasizing automation, continuous integration, and continuous delivery.
     - As an SRE, I not only support deployments but also write code to improve observability, automate toil, and maintain system scalability and resilience.

#### SRE Principles (Simplified)

1. **Embrace risk** : 100% reliability isn’t the goal — aim for “just reliable enough” using SLOs and error budgets.
2. **Service Level Objectives (SLOs)** : Define target reliability levels (e.g., 99.9% uptime), based on user expectations.
3. **Error Budgets** : Allow controlled risk: if you're within budget, release new features; if you're over, focus on stability.
4. **Eliminate toil** : Reduce repetitive manual work (like on-call tasks or deployments) through automation.
5. **Monitoring and observability** : Build systems with strong metrics, logs, and traces so you can understand and fix issues quickly.
6. **Incident response** : Have clear, fast processes for handling outages: alerting, playbooks, postmortems, and blameless culture.
7. **Blameless postmortems** : Focus on learning, not blame — root causes, not human error.
8. **Release engineering** : Manage reliable, automated CI/CD pipelines that balance velocity with safety.
9. **Capacity planning & scalability** : Plan ahead for traffic growth, manage resource usage, and prevent bottlenecks.
10. **Software engineering for ops** : Treat operations problems like coding problems — write tools, scripts, and automation to fix issues at scale.

#### DevOps Principles (Simplified)

1. **Collaboration & Communication** : Break silos between dev, ops, QA, and security teams — work together from start to finish.
2. **Automation** : Automate everything: builds, tests, deployments, infrastructure, and monitoring to move faster and reduce errors.
3. **Continuous Integration (CI)** : Developers integrate code frequently; each change is automatically tested and validated.
4. **Continuous Delivery/Deployment (CD)** : Deliver code to production (or near-production) quickly and reliably, using automated pipelines.
5. **Infrastructure as Code (IaC)** : Manage infrastructure with code (e.g., Terraform, Ansible), so it’s versioned, testable, and repeatable.
6. **Monitoring & Feedback** : Use real-time metrics, logs, and alerts to detect issues early and continuously improve systems.
7. **Shift Left** : Test, secure, and validate early in the development lifecycle, not just after code is written.
8. **Fail Fast, Recover Faster** : Encourage quick experiments, detect failures early, and recover quickly using automated rollbacks and strong incident response.
9. **Security as Code (DevSecOps)** : Build security into the pipeline — automated scans, compliance checks, and secrets management.
10. **Customer-Centric Action** : Focus on user experience; deliver features and fixes that add real value quickly and reliably.

#### SRE - Day-to-Day Tasks
- Role and Responsibilities
1. Incident Management & On-Call
     - Respond to alerts and outages.
     - Perform root cause analysis (RCA) after incidents.
     - Write and review blameless postmortems.
     - Implement follow-up action items to prevent recurrence.
2. Monitoring, Alerts & Observability
     - Define and tune SLIs, SLOs, and alerts.
     - Maintain dashboards (e.g., with Prometheus, Grafana, Datadog).
     - Improve system observability (logs, traces, metrics).
3. Automation & Toil Reduction
     - Identify repetitive manual tasks (toil) and automate them using scripts, tools, or platforms.
     - Develop self-healing and auto-scaling solutions.
4. CI/CD & Deployment Management
     - Manage and improve build and release pipelines.
     - Perform safe rollouts and canary deployments.
     - Implement rollback strategies and deployment validation.
5. Infrastructure & Configuration Management
     - Use Infrastructure as Code (IaC) tools like Terraform or Ansible.
     - Provision and maintain cloud or on-prem infrastructure.
     - Ensure systems are secure, scalable, and cost-efficient.
6. Capacity Planning & Reliability Reviews
     - Analyze system usage trends and plan for growth.
     - Participate in production readiness reviews (PRRs) for new services.
     - Define error budgets and enforce them in release decisions.
7. Cross-Team Collaboration
     - Work closely with developers to improve system reliability.
     - Advocate for best practices in operational excellence.
     - Participate in architecture reviews and design discussions.
8. Documentation & Runbooks
     - Maintain runbooks and incident playbooks.
     - Write internal documentation for services, tools, and SRE processes.

#### Monitoring vs Observability
- Monitoring tracks known metrics and alerts when something goes wrong. Observability provides the ability to understand and diagnose unknown problems by analyzing metrics, logs, and traces together.

- **Observability** is the ability to understand what's happening inside a system just by looking at its outputs
1. It’s about answering “why”
     - While monitoring tells you something is wrong, observability helps you figure out why it's wrong, even for unknown or complex issues.
2. Built on the ‘Three Pillars’
     - **Metrics**: high-level health indicators
     - **Logs**: detailed event records
     - **Traces**: show how requests flow through services
3. Helps debug distributed systems
     - Especially useful in microservices and cloud-native environments where traditional monitoring falls short.
4. Proactive and diagnostic
     - With good observability, you can spot patterns, detect anomalies, and even prevent incidents before they impact users.
5. Driven by instrumentation
     - Services must be instrumented to expose internal state; I often use tools like OpenTelemetry, Grafana, or Elastic Stack to support this.
6. Improves reliability and user trust
     - It’s critical for fast incident response, RCA, and overall system understanding — which directly ties into SRE goals.

- **Monitoring** is the practice of collecting, visualizing, and alerting on system and application health indicators to detect issues early and ensure reliability
1. Detects known issues
     - Monitoring helps catch problems like high CPU, memory usage, error rates, or service downtime — things we can anticipate.
2. Based on predefined metrics
     - Metrics are collected from systems (e.g., latency, throughput, availability) and thresholds are set for alerting.
3. Enables alerting and incident response
     - When metrics breach thresholds, alerts notify the team for immediate action.
4. Reactive and real-time
     - Monitoring is primarily reactive — it's designed to quickly catch and respond to issues when they happen.
5. Supports SLO tracking
     - Metrics from monitoring feed into SLIs/SLOs, helping ensure service reliability goals are met.
6. Tools I’ve used
     - I’ve worked with Prometheus + Grafana, Datadog, CloudWatch, and New Relic to build monitoring solutions.
7. Baseline for observability
     - Monitoring is a critical foundation, but observability extends it by helping debug why things went wrong, not just what went wrong.

#### MTTR - Mean Time to Recovery
- Mean Time to Recovery (MTTR) is the average time it takes to fully restore a service after an incident or failure. It measures how quickly a system can recover from a disruption.
- Measures Reliability & Operational Efficiency
     - Lower MTTR = Faster Recovery = Higher Availability.
- Helps Identify Weaknesses in Incident Response
     - If MTTR is high, investigate slow alerting, debugging, or manual processes.
- Drives Automation & Self-Healing
     - Automating recovery actions (e.g., auto-scaling, rollbacks) reduces MTTR.
- Improves SLAs & Customer Satisfaction
     - Faster recovery means fewer service disruptions for users.
 
#### MTTR - Mean Time to Resolve
- To reduce MTTR (Mean Time to Resolve):
     - Automated Remediation: Use Runbooks and self-healing mechanisms (e.g., Kubernetes auto-restarts).
     - Incident Response Playbooks: Maintain structured response guides for various failures.
     - Blameless Postmortems: Continuous learning from failures to prevent recurrence.
     - ChatOps Integration: Automate incident response using Slack, Microsoft Teams, or PagerDuty.

#### MTTD - Mean Time to Detect
- To reduce MTTD (Mean Time to Detect):
     - Proactive Monitoring: Implement Prometheus, Grafana, Datadog, or Splunk to detect anomalies in real-time.
     - Distributed Tracing: Use Jaeger or OpenTelemetry for tracing requests across microservices.
     - AI/ML-based anomaly detection: Automate log and metric analysis with AI-based solutions like Azure Monitor or AWS DevOps Guru.

#### SLO, SLA, SLI & Error Budget
- I start by understanding user expectations and critical service metrics (SLIs). Then, I set achievable SLOs based on data and business needs. From there, I calculate an error budget to balance innovation and reliability. SLAs formalize commitments externally. I implement real-time monitoring to track SLIs and error budget consumption, enabling proactive incident response and continuous improvement.

1. **SLA (Service Level Agreement)** - its a business-level commitments based on SLOs, often with penalties. **External commitment (contract between provider & customer).**
   - Example: "99.95% uptime per month, or a 5% refund."
   - Use: Ensures accountability; failing an SLA can have financial/legal consequences.
2. **SLO (Service Level Objective)** - A target value for an SLI that defines acceptable performance.  **Internal goal (target reliability, e.g., 99.99% uptime).**
   - Example: "99.9% uptime over 30 days" or "p99 latency < 500ms".
   - Use: Sets a goal for reliability and user experience.
3. **SLI (Service Level Indicator)** - A quantitative measure of a system's performance. - the raw metrics - **Measured metric (actual 99.97% uptime recorded).**
   - Example: Response time, uptime percentage, error rate.
   - Use: Helps track how well a service is performing.
4. **Error Budget** - The allowable margin of failure within an SLO.
   - Example: If the SLO is 99.9% uptime, the error budget is 0.1% downtime (about 43.8 minutes per month).
   - Use: Helps balance innovation and reliability.
   - Error budget defines the acceptable level of failure before SLO violations.
![image](https://github.com/user-attachments/assets/7b93fc84-a951-417f-8190-2863db298798)

![image](https://github.com/user-attachments/assets/44042a63-ef17-43e9-96d7-877fdd1d7b7c)

**Metrics :**
1. Uptime/Availability (e.g., 99.9% availability)
2. Response Time (e.g., incidents responded to within 15 minutes)
3. Resolution Time (e.g., critical issues fixed within 4 hours)
4. Performance Metrics (e.g., API response time < 200ms)
5. Error Rates (e.g., < 0.01% failure rate)
6. Support & Escalation Process

#### SLO dashboards
- In Grafana, I build SLO dashboards by querying SLIs like error rate, latency, and availability from Prometheus. I use gauges and graphs with clear thresholds to visualize if we’re meeting targets. I also track error budget burn rate to proactively detect risk of SLO breach. This helps teams maintain reliability and prioritize incidents effectively.
1. Data Sources : Commonly Prometheus, Loki, or any monitoring system storing your SLIs.
     - SLIs include:
          - Error rate (e.g., HTTP 5xx rate)
          - Latency (e.g., 95th percentile response time)
          - Availability (successful request ratio)
2. Panels to Include
     - SLO Compliance Gauge: Shows % of SLO achieved (e.g., current error budget remaining).
     - Error Rate Graph: Errors over time, with threshold lines.
     - Latency Histogram: Response time percentiles compared to targets.
     - Availability Over Time: Uptime % graph for the service.
     - Error Budget Burn Rate: How fast the error budget is being consumed.
3. Example Metrics Queries
     - Error Rate (Prometheus) : sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))
     - Latency 95th percentile: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
     - Availability: 1 - (sum(rate(http_requests_total{status=~"5.."}[1h])) / sum(rate(http_requests_total[1h])))
4. Thresholds and Alerts
     - Set visual thresholds on panels:
          - Green if SLO met (e.g., error rate < 0.1%)
          - Yellow if nearing breach
          - Red if breached
     - Integrate with alerting (Alertmanager) for SLA/SLO violations.
5. Design Tips
     - Use clear labels for SLO targets and current values.
     - Display error budget remaining prominently.
     - Combine multiple SLIs on one panel for quick glance.
     - Use time ranges (last 30 mins, 1h, 24h) for context.

#### Reduce toil
- By automating repetitive manual tasks, improving monitoring, building self-healing systems, and developing tools that prevent or simplify operational work.

#### Incident Management
- Quickly respond to alerts, follow runbooks/playbooks, communicate clearly, perform root cause analysis after the incident, write blameless postmortems, and implement follow-up improvements

#### Change Management
- Change Management is a structured process used to handle all changes to a system or infrastructure in a controlled and systematic way, minimizing risk and service disruption while ensuring traceability and accountability.
- Change Management Important
     - To reduce outages or incidents caused by unplanned or poorly implemented changes.
     - To maintain system stability while allowing continuous improvement and innovation.
     - To ensure compliance, auditability, and communication among stakeholders.
- Types of Changes
1. **Standard Changes** :Pre-approved, low-risk, routine changes that follow a defined process without needing additional approval each time.
     - Software patching from a tested update
          - Adding users or permissions
          - Scheduled backups or minor configuration tweaks
     - Used for routine operational tasks that are well-understood and unlikely to cause issues.
2. **Normal Changes** : Changes that require risk assessment, testing, and formal approval before implementation.
     - These can be low, medium, or high risk and usually follow a Change Advisory Board (CAB) review.
          - Deploying new features to production
          - Infrastructure upgrades
          - Changes to critical configurations
     - Used when the impact is uncertain or the change could affect service availability or security.
3. **Emergency Changes** :Changes that must be implemented immediately to resolve a critical incident or security vulnerability.
     - Typically bypass some standard approvals but require post-implementation review.
          - Patching a critical security flaw
          - Rolling back a failed deployment
          - Fixing a production outage
     - Used only when immediate action is necessary to restore service or security.

#### Capacity planning
- Capacity planning predicts future resource needs to prevent outages and ensure performance during load spikes. It helps avoid over-provisioning (wasting cost) and under-provisioning (causing failures).
