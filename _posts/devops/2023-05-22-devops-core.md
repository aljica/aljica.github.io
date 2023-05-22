---
layout: post
title: Core DevOps Concepts
categories: devops
---

# COMPONENTS OF DEVOPS
	• CI/CD
	• Continuous monitoring/improvement
	• IaC
	• Configuration management

![Image](/docs/assets/images/devops/devops-core/1.png)

![Image](/docs/assets/images/devops/devops-core/2.png)
	
# CONTINUOUS INTEGRATION / CONTINUOUS DEPLOYMENT
	• Continuous Integration (test DEV's code)
		○ Automate code testing
			1) Dev pushes code
			2) Automated build
			3) Automated unit/integration testing
	• Continuous deployment (deploy to PROD)
		1) Dev pushes code
		2) Automated build
		3) Automated unit/integration testing
		4) Additional manual tests
		5) Provision resources
			§ IaC scripts
		6) Deploy
			• IaC scripts
		7) Monitor
		8) Rollback if necessary
			• Version control

![Image](/docs/assets/images/devops/devops-core/3.png)

# CONTINUOUS MONITORING/IMPROVEMENT METRICS/KPI
	• Change volume
		○ Number of new lines of code
		○ Number of bug fixes
	• Deployment frequency
		○ How often a team is deploying
		○ Should be consistent if mature software, upward trend if new
	• Time from starting dev to deployment
	• % of failed deployments
		○ Review in conjunction with change volume!
	• Availability
		○ How many releases led to => violations of SLA?
		○ Average downtime?
	• Customer complaint volume
	• % change in user volume (sign-ups, log-ins)

# IaC
	• Template-driven deployment
	• Implement the following in an organization:
		○ Standardization
		○ Consistency
		○ Compliance

# CONFIGURATION/CHANGE/PATCH/VULN MANAGEMENT
	• Chef/Puppet/Ansible (tools to manage IaC)
		○ Automate sysadmin tasks including:
			▫ Provisioning of resources
			▫ Configuration & management of IT resources
	• Increase productivity
		○ Deploy to hundreds of resources simultaneously
	• Decrease downtime (IMMUTABLE DEPLOYMENT) (PATCH/VULN MGMT)
		1) Update configuration on a subset of PROD machines
		2) Test live against a subset of users
		3) Gradually delete existing PROD machines and replace them with new machines with the new configuration
		4) Monitor this process continuously

# TRADITIONAL ORGANIZATIONS
	• Disconnect between operations (administrators) and developers
		○ They do not understand each other's needs
		○ Might use different tools/processes
			§ Devs might run their code on a certain OS build, whereas Ops 
            might deploy on a different, incompatible OS build

# BENEFITS OF DEVOPS
	• Speed
		○ Quick to new releases
		○ Adapt to changing needs
	• Fast delivery
		○ Automating end-to-end pipelines (dev, build, deployment)
	• Reliability
		○ CI/CD for automated security testing
	• Scalability
	• Collaboration
		○ Shared responsibility model between Dev & Ops
	• Security
		○ Automate security checking of the frequent changes

# INTRODUCING DEVSECOPS
	• Security automation & at scale

	1) Code: Scan all code for secret/access keys
	2) Build: Tag security artifacts e.g. access tokens, encryption keys for easy ID
	3) Test: Scan code for vulnerabilities, test security standards
	4) Deploy: All security components should be in place. Checksums
	5) Monitor: security standards, vulnerable dependencies, changing compliance regulations

#### AppSec Testing
	• Software composition analysis (SCA)
		○ Check for known vulns in dependencies (Snyk)
	• SAST
		○ SonarQube, during coding (SonarLint)
	• DAST
		○ Web app vuln scanner
		○ Black-box pentest against live system
	• IAST
		○ App is run by automated test; simultaneously run vulnerability tests

![Image](/docs/assets/images/devops/devops-core/4.png)

# IMPLEMENTING A CD STRATEGY
	• In-place deployment
		○ Deploy in one big sweet
		○ Full rollback only option to restore
		○ Low cost, fast to deploy
	• Rolling deployment
		1) Divide servers into groups
		2) Deploy to 1 group at a time
		○ Achieve zero down-time, easy rollback
	• Blue-green deployment
		○ Blue env: PROD
		○ Green env: new environment mimicking PROD
		○ Gradually divert more and more of 
PROD traffic from blue to green (load balancing/canary analysis).
2 methods:
			1) DNS cutover
			2) Autoscaling
				• Replace existing servers with servers running
new software version when autoscaling.
		○ Zero downtime, easy rollback
	• Red-black deployment
		1) Canary testing: divert 1% of prod traffic to new red environment
		2) If all is OK, proceed.
		3) Set resources for red environment based on current usage in PROD
		4) DNS cutover from black (old PROD env) to new red env
		5) Rollback: return DNS to black (old PROV env)
		6) Use as Beta
	• Immutable deployment
		○ Tear some servers down, spin up new ones, etc.
		○ Good for older app infrastructures with many patches over time

# Continuous testing in CI/CD pipeline
	• Unit tests 
		○ 70% of testing
		○ Conducted on programmer's PC
		○ Detect bugs
		○ Check: Code coverage, adherence to coding guidelines etc.
	• Integration/System testing
		○ Testing environments that mimic PROD
		○ Costlier
		○ Different testing teams
		○ Regression testing (code coverage)
	• User Acceptance Testing (UAT)
		○ Replica of PROD
		○ System/performance testing
	• A/B & Canary testing
		○ Final testing conducted in PROD
		○ A/B & Canary Testing
			§ Divert % to new PROD systems
			§ Ask users what they think
			§ Collect feedback
			§ Adjust new PROD system accordingly
	• Load testing of front-end, DB etc.

# Testing & Deploying new builds
	1) Pull code from git
	2) Integrate to main branch
	3) Build code (Jenkins)
	4) Choose a deployment method (Jenkins)
		• OneAtATime
		• HalfAtATime
		• AllAtOnce
		• Custom
	5) Deploy code according to the chosen method
		1) Stop running application
		2) Download application binary
		3) BeforeInstall: create backups
		4) Install (Ant/Maven for Java apps)
		5) Post-install config (log mgmt etc)
		6) AppStart
		7) Validate/Verify (monitor)

![Image](/docs/assets/images/devops/devops-core/5.png)

![Image](/docs/assets/images/devops/devops-core/6.png)

![Image](/docs/assets/images/devops/devops-core/7.png)

![Image](/docs/assets/images/devops/devops-core/8.png)