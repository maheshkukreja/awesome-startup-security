# Awesome Startup Security [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of security resources for startup founders who'd rather be building product.

Security isn't your priority - shipping is. But security failures kill startups in specific, preventable ways: a leaked AWS key burns through your cloud credits overnight, a breach loses your first enterprise deal, a missing SOC 2 report stalls the sales pipeline for months. This list tells you what to do, when to do it, and why you'd actually care - based on what practitioners who've built security at startups actually recommend.

**Jump to your stage:** [Just me and my cofounder](#just-me-and-my-cofounder-building-the-mvp) · [5-15 people](#we-have-a-product-and-5-15-people) · [Enterprise customers asking about SOC 2](#enterprise-customers-are-asking-about-soc-2-15-40-people) · [40+ people](#scaling-40-people) · [SOC 2 prep checklist](#the-soc-2-prep-checklist)

## Where to Start - The Founder's Journey

Most security advice is written for companies with dedicated teams and six-figure budgets. This is advice for you - organized by business stage, not security maturity level. Every recommendation here passes the test: "Would a founder with no security background actually do this, and why?"

**Our philosophy:** Security should scale with business risk, not fear. The goal isn't to be maximally secure — it's to reduce the specific risks that kill startups at each stage. Every recommendation in this guide earns its place by being worth a founder's limited time.

> **A note on pricing:** Free tiers, pricing, and usage limits mentioned throughout this guide were accurate at time of writing — always verify current terms directly with vendors before making decisions.

One principle applies at every stage: **collect less data.** The cheapest security control is not storing sensitive data you don't need. Every field you don't collect is a field that can't be breached.

### Just Me and My Cofounder, Building the MVP

You have no revenue, no security budget, and that's fine. These items protect you from the things that would actually set you back. The goal at this stage is minimizing blast radius, not eliminating risk.

1. **Google Workspace (Business Starter, roughly $7/user/month on annual plans at time of writing).** You need a custom domain email to look credible to customers. That's the real reason to buy it. What you get for free with it: MFA (TOTP is fine to start), Google Sign-In for SaaS tools, and the foundation for SSO later. This is the single most cost-effective security decision you'll make.

2. **SPF/DKIM/DMARC for your email domain.** Free, set up once when you configure Google Workspace. Stops people from spoofing your domain to send phishing emails to your customers. Takes 30 minutes.

3. **Private repos on GitHub + MFA on your accounts.** Your source code is your company. Set repos to private and turn on MFA for every GitHub account. This is a realistic security action because you already care about your code - you're just making sure nobody else can get to it.

4. **Keep secrets out of your code.** Put API keys and cloud credentials in [GitHub Actions secrets](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions), not in code or config files. Even with private repos, secrets in code are risky because: (a) repos can accidentally be made public, (b) CI/CD logs may expose env vars, (c) anyone with repo access sees all secrets, (d) git history preserves secrets even after deletion. GitHub Actions secrets are access-controlled and redacted from logs when used correctly, but workflows can still leak sensitive values if you print them or mishandle outputs. **5 minutes of setup saves you $10K+ in cloud bills.**

5. **Enforce HTTPS.** If you're deploying to a VPS or managing your own server, use [Let's Encrypt](https://letsencrypt.org/) (free, automated TLS certificates) or [Caddy](https://caddyserver.com/) (automatic HTTPS by default). Most managed platforms (Vercel, Railway, Render, Cloudflare Pages) handle this for you. For internet-facing products, HTTPS should be the default.

6. **Your browser's built-in password manager.** You don't need 1Password yet - you have no revenue. Chrome's built-in [Google Password Manager](https://passwords.google.com/) or Firefox's built-in password manager generates strong random passwords, syncs across devices, and alerts you to breached passwords. It's free and it's already there. Use it for every site that doesn't support Google Sign-In. (When anyone on your team needs to share a credential, move to a dedicated password manager — that's when passwords end up in Slack.)

7. **Full-disk encryption.** Macs with Apple Silicon or T2 chips encrypt storage by default, and FileVault adds stronger login-bound protection. On Windows, device encryption/BitLocker may be enabled automatically on supported devices — verify it. On Linux, enable full-disk encryption during install if you need it.

8. **Turn on Dependabot.** Free, takes 30 seconds in GitHub settings. It'll flag vulnerable dependencies in your code. Will the alerts pile up unread? Probably. That's okay - when something critical hits (like Log4Shell), you'll at least get the notification.

9. **If you're using AWS, turn on CloudTrail early.** The first copy of management events in each Region is free, but additional management-event copies and data events cost money. You still want it on because you can't investigate much without an audit trail.

10. **Automated database backups.** If your database gets corrupted or deleted, your company dies. Your managed database provider (RDS, Supabase, PlanetScale) likely has automated backups — verify they're enabled and that you know the retention period. If you're self-hosting Postgres or MySQL, set up `pg_dump` or equivalent on a cron and store backups offsite (S3 with versioning). This takes 15 minutes and is non-negotiable.

11. **Know who controls your domain, DNS, and cloud billing.** Founders have lost companies to expired domains, hijacked registrars, and suspended AWS accounts. Make sure at least two founders have access to: your domain registrar, your DNS provider, your cloud billing account, your GitHub org, and your Google Workspace admin. If any one person controls a critical account with no backup admin, that's a bus-factor-one risk to your company.

**Don't worry about yet:** SOC 2, a security hire, SIEM, pentesting, bug bounty, MDM, dedicated password managers, or protected branches. You're 1-2 people committing to main - that's fine. Almost anything goes pre-PMF.

### We Have a Product and 5-15 People

You have paying customers and a growing team. Security starts mattering because losing customer data or access would hurt your business, and people are joining and leaving.

1. **Protected branches + PR reviews.** You now have enough people to review each other's code. Lock your main branch, require at least one approval. This is as much about code quality as security.

2. **Team password manager.** [Bitwarden](https://bitwarden.com/) supports free sharing for two people; paid Teams plans start at per-user pricing. [Vaultwarden](https://github.com/dani-garcia/vaultwarden) is a self-hosted option. You need shared vaults now because your team shares credentials for AWS, your domain registrar, SaaS tools, and more. Browser password managers can't do shared credentials - that's when people start pasting passwords in Slack, which is how they leak.

3. **Start looking at Dependabot alerts.** Assign someone to triage the critical ones monthly. You don't need to fix everything - just the high-severity ones that affect dependencies you actually use.

4. **SSO for critical SaaS via Google Workspace.** Connect your most important tools (GitHub, Slack, your cloud provider) to Google Workspace via SAML/OIDC. When someone leaves, you disable one Google account and they lose access to everything connected. Don't pay the [SSO tax](https://sso.tax/) on every tool - be selective about which ones justify the enterprise tier.

5. **Basic cloud guardrails.** Don't leave S3 buckets public. Use IAM roles instead of root keys or static credentials. If you're on AWS, [turn on CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html) if you haven't already - it's your audit trail, and it's non-negotiable according to people who do this for a living.

6. **Don't put production secrets in `.env` files on your server.** `.env` files are fine for local development, but in production they're unencrypted, unaudited, and anyone with SSH access can read them. As you move past simple PaaS deployments or shared credentials, move production secrets into a real secrets manager (AWS Secrets Manager, GCP Secret Manager, [Doppler](https://www.doppler.com/), [Infisical](https://infisical.com/), etc.).

7. **Onboarding/offboarding checklists.** Write down what accounts new hires need and what gets revoked when someone leaves. This seems tedious until the first time someone departs and you spend a week trying to figure out what they had access to.

**Start thinking about:** your [first security hire](https://ramimac.me/start-security), what to say when sales prospects send security questionnaires, and whether enterprise customers are asking about SOC 2.

### Enterprise Customers Are Asking About SOC 2 (15-40 People)

The trigger for this stage isn't headcount - it's when a deal stalls because a customer asks "Are you SOC 2 compliant?" and your sales team doesn't have an answer. Security is now a revenue blocker.

1. **Hire your first security person (employee #20-40).** Your first security hire is usually a hands-on senior security engineer, not an executive security leader. Willingness to accept a "CISO" title at a small startup is actually [a mild red flag](https://news.ycombinator.com/item?id=32667175). Read the [Hiring & Building Security Teams](#hiring--building-security-teams) section.

2. **SOC 2 Type I.** Don't pursue SOC 2 because it feels responsible - do it because a customer requires it. SOC 2 is documentation, not reality. Audits are performed by accountants, not pentesters. Use [Vanta](https://www.vanta.com/), [Drata](https://drata.com/), or [Secureframe](https://secureframe.com/) to automate the painful parts. **Scope your SOC 2 narrowly - only include systems that customers interact with. You can always expand scope later. A narrow scope means fewer controls, less evidence, and a faster audit.** Note: Pentesting is not required for SOC 2 Type I - it's helpful but not mandatory. That said, enterprise procurement teams often demand a recent pentest alongside your Type I report — the framework doesn't require it, but the customer might.

3. **Centralized logging.** You probably already have Datadog, Grafana, or CloudWatch for ops. Make sure security-relevant events (logins, permission changes, API errors) go there too. You can't investigate what you can't see.

4. **Write an incident response plan.** Use the [PagerDuty template](https://response.pagerduty.com/) - it's free and battle-tested. Takes an afternoon. Do a tabletop exercise with your team. Before this stage, "call the CEO" was your incident response plan, and that was fine.

5. **Secrets scanning in CI.** Add [Gitleaks](https://github.com/gitleaks/gitleaks) to your CI pipeline. You turned on Dependabot ages ago but probably weren't triaging it. Now you have someone who will. The cost of a leaked credential is much higher now that you have real customers.

6. **Infrastructure as Code.** Move cloud provisioning into Terraform (or Pulumi/CDK) and keep configs in Git with the same PR process as code. The AWS console is the enemy of auditability - and your SOC 2 auditor will want to see change management. Store Terraform state in a remote backend (S3, GCS, Terraform Cloud) with versioning enabled — losing your state file means losing track of what's deployed.

7. **Test your backups.** Restore a database backup to a test environment at least once. Backups that have never been tested are not backups — they're hopes.

8. **Create a `/security` page on your website.** List what you do: MFA enforcement, encryption at rest and in transit, incident response contact, compliance status. This reduces inbound security questionnaires by 30-50% because prospects can self-serve. [SafeBase](https://safebase.io/) hosts these as a product; you can also just build a static page.

9. **Security questionnaire automation.** Enterprise sales often means drowning in security questionnaires. Tools like [Conveyor](https://www.conveyor.com/), [SafeBase](https://safebase.io/), or Vanta's questionnaire automation can cut response time from days to hours. This becomes relevant once you're fielding more than a few questionnaires per quarter.

10. **Data Processing Agreement (DPA).** Enterprise deals often require a DPA alongside SOC 2. Have your lawyer draft a template DPA — you'll reuse it for every enterprise customer. If you're handling EU data, this is legally required under GDPR.

**Don't worry about yet:** SOC 2 Type II, pentesting, bug bounty, MDM, or a security team. One strong security engineer can handle all of this for most B2B SaaS startups. If you're in fintech, healthtech, or handling large volumes of sensitive data, you may need specialized expertise sooner.

### Scaling (40+ People)

Security is now a business function, not a side quest.

1. **SOC 2 Type II.** Your Type I was point-in-time. Type II proves you maintained controls over a period (usually 12 months). Enterprise customers increasingly require it.

2. **First pentest.** Budget $25K-35K for a SaaS product pentest. Do it after your product is relatively stable, not while it's changing weekly. Annual cadence or on major releases.

3. **Build the security team.** Second security engineer, then a Director of Security, then specialists (product security, security operations, IT security). The CISO hire comes much later - [it's often a "marquee resume" hire to impress the market](https://news.ycombinator.com/item?id=32667175).

4. **Endpoint security / MDM.** If you haven't already, start with lightweight device visibility (Google Workspace endpoint verification, [Kolide](https://www.kolide.com/) paired with Slack/Okta) before jumping to full MDM (Jamf). Full MDM makes sense now, but only if you have someone to manage it. [MDM without an operator is security theater.](https://news.ycombinator.com/item?id=32667865)

5. **Bug bounty.** Start with [HackerOne](https://www.hackerone.com/) or [Bugcrowd](https://www.bugcrowd.com/). You need a mature enough product and a team ready to triage and fix reports.

6. **Security champions.** Embed security knowledge in every engineering team with a [security champions program](https://owasp.org/www-project-security-culture/stable/4-Security_Champions/). This scales better than a centralized security team reviewing everything.

### The SOC 2 Prep Checklist

This practical checklist was originally developed by [Latacora](https://www.latacora.com/blog/2020/03/12/soc2-starting-seven/) and updated based on Thomas Ptacek's experience [taking Fly.io through SOC 2](https://news.ycombinator.com/item?id=32667865):

| # | Item | When | Notes |
|---|------|------|-------|
| 1 | Single Sign-On (SSO) | 5-15 people | Google Workspace + Tailscale is a [practitioner-recommended combination](https://news.ycombinator.com/item?id=32667865) |
| 2 | PRs, Protected Branches, CI/CD | 5+ people | When you have enough people to review code |
| 3 | Centralized Logging | 15+ people | You probably already have this for ops |
| 4 | Terraform (or Something) | 15+ people | IaC = auditability for SOC 2 |
| 5 | CloudTrail and AssumeRole | Day 1 on AWS | "You can't deploy on AWS and not have an audit trail" - tptacek |
| 6 | ~~MDM~~ | After first security hire | Ptacek now recommends deferring. Endpoint security has fallen out of fashion in SOC 2 audits. |
| 7 | Vendor Security | 15+ people | Start a spreadsheet of every SaaS tool you use |

## Contents

- [Getting Started](#getting-started)
- [Compliance & Frameworks](#compliance--frameworks)
- [Application Security](#application-security)
- [Cloud Security](#cloud-security)
- [Identity & Access Management](#identity--access-management)
- [Infrastructure Security](#infrastructure-security)
- [Security Operations](#security-operations)
- [Supply Chain Security](#supply-chain-security)
- [AI/ML Security](#aiml-security)
- [Books](#books)
- [Videos & Talks](#videos--talks)
- [Newsletters & Blogs](#newsletters--blogs)
- [Security Vendor Programs for Startups](#security-vendor-programs-for-startups)
- [Hiring & Building Security Teams](#hiring--building-security-teams)
- [Incident Response](#incident-response)
- [Further Reading](#further-reading)
- [Contributing](#contributing)

---

## Getting Started

**Why you'd care:** These guides save you from wasting time on the wrong things. Most security advice is for enterprises - these are specifically calibrated for startups. **When:** Before you do anything else on this list. **First step:** Read the Starting Up Security guide - it takes 20 minutes and gives you a framework for every security decision.

- [Starting Up Security](https://scrty.io/) - Ryan McGeehan's practical guide on right-sizing security for startups at different stages. The best starting point for any founder.
- [The SOC2 Starting Seven](https://www.latacora.com/blog/2020/03/12/soc2-starting-seven/) - Latacora's 7 things that immediately improve security *and* simplify a future SOC 2 audit. Do items 1-5 and 7; defer MDM.
- [Minimum Viable Secure Product (MVSP)](https://mvsp.dev/) - A minimum security baseline checklist for B2B software. When your customers start sending security questionnaires, start here.
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - The 10 most critical web application security risks. Every developer should know these - not because of compliance, but because they're the actual ways products get hacked.
- [CIS Controls](https://www.cisecurity.org/controls) - Prioritized security actions organized by implementation group. IG1 is the relevant starting point for startups (IG2/IG3 are enterprise-focused).
- [Secure by Design (CISA)](https://www.cisa.gov/securebydesign) - CISA's guidance on building security into products from the start.
- [Google's Security First Principles](https://cloud.google.com/security/overview/whitepaper) - Google Cloud's security whitepaper covering foundational security design principles.
- [Mozilla Web Security Guidelines](https://infosec.mozilla.org/guidelines/web_security) - Practical web security guidelines covering headers, TLS, CSP, and more. Battle-tested and actionable.
- [GitLab Security Handbook](https://handbook.gitlab.com/handbook/security/) - GitLab's public security handbook. A real-world example of how a company documents and operates its security program.
- [The SaaS CTO Security Checklist](https://www.goldfiglabs.com/guide/saas-cto-security-checklist/) - A practical security checklist for SaaS CTOs. Organized by priority with clear action items.

## Compliance & Frameworks

**Why you'd care:** Compliance doesn't make you secure, but it unblocks revenue. Enterprise customers require SOC 2 before signing contracts. Your sales team may already be making things up on security questionnaires. **When:** When a customer asks "Are you SOC 2 compliant?" - not before. **First step:** Read the Latacora Starting Seven. Those items generate SOC 2 evidence as a side effect of being useful.

**The reality of SOC 2:** Audits are performed by accountants, not pentesters. You tell your auditor what controls you have. They verify you documented them. They give you a report. SOC 2 verifies that you documented your controls and followed them — it doesn't verify that your security is actually strong. But it's what your customers require and your competitors already have.

### SOC 2

- [SOC 2 Compliance Guide by Vanta](https://www.vanta.com/collection/soc-2) - Comprehensive guide to SOC 2 requirements, trust service criteria, and the audit process.
- [SOC 2 Starter Guide by Drata](https://drata.com/blog/soc-2-compliance-guide) - Practical walkthrough from scoping to audit readiness.
- [AICPA SOC 2 Overview](https://www.aicpa-cima.com/topic/audit-assurance/audit-and-assurance-greater-than-soc-2) - The official AICPA resource on the SOC 2 framework.

### ISO 27001

*Relevant when selling to European enterprises or when customers specifically require it. Most US-focused startups start with SOC 2.*

- [ISO 27001 Overview](https://www.iso.org/standard/82875.html) - Official ISO page for the ISO/IEC 27001:2022 information security standard.
- [ISO 27001 Implementation Guide by Advisera](https://advisera.com/27001academy/what-is-iso-27001/) - Free guide covering the basics of ISO 27001 implementation and certification.

### GDPR

- [GDPR.eu](https://gdpr.eu/) - The most accessible guide to GDPR requirements, in plain English with practical checklists.
- [ICO Guide to GDPR](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/) - The UK Information Commissioner's authoritative guide to GDPR compliance.

### HIPAA

*Only relevant if you handle Protected Health Information (PHI). If you're not in healthcare, skip this.*

- [HHS HIPAA for Professionals](https://www.hhs.gov/hipaa/for-professionals/index.html) - Official U.S. government HIPAA guidance for covered entities and business associates.
- [HIPAA Security Rule Guidance by NIST](https://csrc.nist.gov/pubs/sp/800/66/r2/final) - NIST SP 800-66 Rev 2: implementing the HIPAA Security Rule with practical guidance.

### PCI DSS

*Only relevant if you process, store, or transmit credit card data directly. If you use Stripe or a similar payment processor, they handle PCI compliance — you just need to fill out an SAQ-A.*

- [PCI DSS Quick Reference Guide](https://www.pcisecuritystandards.org/document_library/) - The PCI Security Standards Council's document library, including quick reference guides and SAQs.

### Multi-Framework

- [NIST Cybersecurity Framework (CSF) 2.0](https://www.nist.gov/cyberframework) - Common language for managing cybersecurity risk. Free, widely adopted, and useful for structuring conversations with security-aware customers.
- [SOC 2 vs ISO 27001 Comparison by Vanta](https://www.vanta.com/collection/soc-2/iso-27001-vs-soc-2) - Helps you decide which compliance framework to pursue first (hint: SOC 2 in the US, ISO 27001 internationally).

## Application Security

**Why you'd care:** Your code is your primary attack surface. A vulnerability in your app is how customer data gets stolen, which kills trust and kills deals. **When:** Day 1 for Dependabot. Gitleaks in CI when you have a pipeline. SAST tools come later when you have someone to act on findings. **First step:** Turn on Dependabot. Takes 2 minutes.

Use [Gitleaks](https://github.com/gitleaks/gitleaks) in CI for free secret detection. [GitHub Secret Protection](https://github.com/features/security) adds org-wide coverage for private repos but requires a paid plan — evaluate it when you have a security person and budget.

### Guides & References

- [OWASP Application Security Verification Standard (ASVS)](https://owasp.org/www-project-application-security-verification-standard/) - A framework for specifying and testing application security requirements across three verification levels.
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) - Practical security guidance on authentication, cryptography, injection prevention, and more. Bookmark this - you'll reference it every time you build a new feature.
- [OWASP Secure Coding Practices Quick Reference](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/) - Checklist of secure coding practices that can be integrated into your development lifecycle.
- [OWASP SAMM](https://owaspsamm.org/) - Software Assurance Maturity Model. Helps you assess and improve your secure development practices over time.

### Static Analysis (SAST)

- [Semgrep](https://semgrep.dev/) - Lightweight, fast, open-source static analysis. The OSS CLI is free with no limits; write custom rules or use community rulesets. Supports 30+ languages. The cloud platform adds cross-file analysis and team collaboration.
- [CodeQL](https://codeql.github.com/) - GitHub's semantic code analysis engine. Free for open-source projects; integrated into GitHub Advanced Security.
- [Bandit](https://bandit.readthedocs.io/) - Python-specific security linter that finds common security issues.
- [Snyk Code](https://snyk.io/product/snyk-code/) - SAST from the Snyk platform. See also [Snyk](#dependency-scanning) under Dependency Scanning for SCA.
- [Brakeman](https://brakemanscanner.org/) - Static analysis scanner specifically for Ruby on Rails applications.

### Dynamic Analysis (DAST)

- [OWASP ZAP](https://www.zaproxy.org/) - The most widely used open-source web application security scanner. Good for a quick automated scan before a release.
- [Nuclei](https://github.com/projectdiscovery/nuclei) - Fast, template-based vulnerability scanner. Community-driven with thousands of detection templates.
- [Burp Suite Community](https://portswigger.net/burp/communitydownload) - Free version of the industry-leading web security testing tool.

### Dependency Scanning

- [Dependabot](https://github.com/dependabot) - GitHub's automated dependency update tool. Creates PRs for vulnerable dependencies automatically. Free. Turn it on now.
- [Snyk](https://snyk.io/) - Developer security platform for dependency scanning (SCA). Free tier includes limited projects. Also offers SAST via Snyk Code (see [Static Analysis](#static-analysis-sast)). Startup program available.
- [OSV-Scanner](https://osv.dev/) - Google's open-source vulnerability scanner using the OSV database. Supports multiple ecosystems.
- [Socket.dev](https://socket.dev/) - Detects supply chain attacks in npm, Python, and Go packages by analyzing package behavior, not just known CVEs. Free for open-source repos and limited private repos.

### Secrets Detection

- [Gitleaks](https://github.com/gitleaks/gitleaks) - Detects hardcoded secrets in git repos. Easy CI/CD integration. The cost of a leaked AWS key is measured in thousands of dollars.
- [TruffleHog](https://github.com/trufflesecurity/trufflehog) - Finds credentials in git history, S3 buckets, and more. Broad detector coverage.
- [detect-secrets](https://github.com/Yelp/detect-secrets) - Yelp's tool for preventing secrets from entering your codebase.

## Cloud Security

**Why you'd care:** Misconfigured cloud infrastructure is how startups get breached - public S3 buckets, overly permissive IAM roles, and hardcoded credentials are the usual suspects. The consequences are real: unauthorized access to customer data, crypto miners on your account, and a very bad week. **When:** As soon as you have a cloud account - basics like not using root and turning on CloudTrail are covered in the MVP stage above. **First step:** On AWS, turn on CloudTrail. The first copy of management events per Region is free, and you can't investigate much without it.

### Multi-Cloud

- [Prowler](https://github.com/prowler-cloud/prowler) - Open-source security assessment tool for AWS, Azure, and GCP. Run it once and it'll tell you what's misconfigured. Checks against CIS benchmarks.
- [ScoutSuite](https://github.com/nccgroup/ScoutSuite) - Multi-cloud security auditing tool by NCC Group. Supports AWS, Azure, GCP, Oracle, and Alibaba Cloud.
- [CloudSploit](https://github.com/aquasecurity/cloudsploit) - Open-source cloud security configuration scanner by Aqua Security.

### AWS

- [AWS Well-Architected Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html) - AWS's own guide to building secure workloads. Dense but authoritative.
- [AWS Startup Security Baseline (SSB)](https://docs.aws.amazon.com/organizations/latest/userguide/aws-startup-security-baseline.html) - AWS guidance specifically for startups. Designed for teams without a dedicated security person.

### GCP

- [Google Cloud Security Best Practices](https://cloud.google.com/security/best-practices) - Google's recommended security configurations and architecture patterns.
- [GCP Security Foundations Blueprint](https://cloud.google.com/architecture/security-foundations) - Reference architecture for setting up a secure GCP organization from scratch.

### Azure

- [Azure Security Benchmark](https://learn.microsoft.com/en-us/security/benchmark/azure/) - Microsoft's security recommendations mapped to CIS and NIST frameworks.
- [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/) - Cloud security posture management with a free tier covering basic recommendations.

### Infrastructure as Code Security

- [Checkov](https://github.com/bridgecrewio/checkov) - Open-source static analysis for IaC (Terraform, CloudFormation, K8s, Helm, Dockerfiles). 1000+ built-in checks.
- [KICS](https://github.com/Checkmarx/kics) - Supports Terraform, Ansible, Docker, CloudFormation, and more.
- [tfsec](https://github.com/aquasecurity/tfsec) - Static analysis for Terraform code across AWS, Azure, GCP, and more. Note: tfsec is being merged into [Trivy](https://github.com/aquasecurity/trivy); for new projects, use `trivy config` instead.

## Identity & Access Management

**Why you'd care:** When someone leaves your company and still has access to your AWS account, your codebase, and your customer data - that's a business risk, not just a security problem. When your team reuses passwords and one of them shows up in a breach, attackers try those credentials on your systems. **When:** Day 1 for password hygiene and MFA. Post-PMF for SSO. **First step:** Enable MFA on GitHub, Google Workspace, and your cloud provider. Use your browser's built-in password manager for everything else.

**On password managers:** For a solo founder or two cofounders who don't share credentials, your browser's built-in password manager (Google Password Manager or Firefox) is fine. They generate strong random passwords, sync across devices, and check for breached passwords. The gap is team features: no shared vaults, no admin controls, no onboarding/offboarding. When you hit 2+ people sharing credentials, move to [Bitwarden](https://bitwarden.com/) (free sharing for 2 people; paid Teams plans at per-user pricing) or [1Password Business](https://1password.com/business/).

**On MFA:** TOTP apps (Google Authenticator, Authy) are fine to start. Hardware keys (YubiKey or generic FIDO2) are better - people lose phones, and hardware keys are phishing-resistant. WebAuthn/passkeys are the future. But any MFA is dramatically better than no MFA. For your highest-value accounts (email admin, registrar, cloud root/admin, GitHub org owner), prefer passkeys or hardware security keys over SMS or TOTP when available.

- [NIST Digital Identity Guidelines (SP 800-63)](https://pages.nist.gov/800-63-4/) - The authoritative guide to digital identity, authentication, and federation from NIST.
- [Keycloak](https://www.keycloak.org/) - Open-source identity and access management. Provides SSO, OIDC, SAML, and user federation out of the box.
- [Authelia](https://www.authelia.com/) - Open-source authentication and authorization server providing 2FA and SSO via OIDC/OAuth2.
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) - Comprehensive guidance on implementing secure authentication mechanisms.
- [WorkOS](https://workos.com/) - Enterprise-ready SSO and directory sync. Generous free tier available. Built for B2B startups that need to offer SSO to their customers.
- [Clerk](https://clerk.com/) - Developer-first authentication platform with generous free tier. Handles SSO, MFA, and user management.
- [Auth0 by Okta](https://auth0.com/) - Free tier available (currently up to 25,000 MAUs). Widely used by startups.
- [Open Policy Agent (OPA)](https://www.openpolicyagent.org/) - Unified policy engine for authorization across your stack. CNCF graduated project.

## Infrastructure Security

**Why you'd care:** Containers and Kubernetes are powerful, but they're also massive attack surfaces. A misconfigured container or an unpatched base image is how attackers get a foothold. **When:** Once you're running containers or managing your own infrastructure. If you're on a managed PaaS (Heroku, Railway, Render) or purely serverless, skip this section - revisit when your architecture evolves. **First step:** Scan your container images with [Trivy](https://github.com/aquasecurity/trivy). Free, fast, and catches known vulnerabilities before they reach production.

**Common mistake:** Running Kubernetes before you need it. **If you're running Kubernetes before you have product-market fit, security is not your biggest problem.** K8s is complex to secure and operate — stay on a managed PaaS or simple container service as long as you can.

### Container Security

- [Docker Security Best Practices](https://docs.docker.com/engine/security/) - Official Docker documentation on securing containers and the Docker daemon.
- [Trivy](https://github.com/aquasecurity/trivy) - Comprehensive vulnerability scanner for containers, IaC, and git repositories. Fast, easy to use, widely adopted.
- [Grype](https://github.com/anchore/grype) - Open-source vulnerability scanner for container images and filesystems by Anchore.
- [Distroless Container Images](https://github.com/GoogleContainerTools/distroless) - Google's minimal container images containing only your application and its runtime dependencies. Smaller image = smaller attack surface.

### Kubernetes

- [NSA/CISA Kubernetes Hardening Guide](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF) - The U.S. government's comprehensive guide to hardening Kubernetes clusters.
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes) - Industry-standard security configuration benchmark for Kubernetes.
- [Falco](https://falco.org/) - Cloud-native runtime security. Detects anomalous activity in containers and Kubernetes. CNCF graduated project.
- [kube-bench](https://github.com/aquasecurity/kube-bench) - Checks Kubernetes clusters against CIS Kubernetes Benchmark.
- [Kubescape](https://github.com/kubescape/kubescape) - Kubernetes security platform that scans clusters, YAML files, and Helm charts against multiple frameworks.

### Network Security

- [Tailscale](https://tailscale.com/) - Zero-config WireGuard-based VPN. Free for personal use and small teams. Google SSO + Tailscale is a [widely recommended combination](https://news.ycombinator.com/item?id=32667865) for startup network security.
- [Cloudflare Zero Trust](https://www.cloudflare.com/zero-trust/) - Free tier includes Zero Trust access, DNS filtering, and basic gateway.
- [Nmap](https://nmap.org/) - The standard for network discovery and security auditing.

## Security Operations

**Why you'd care:** You can't fix what you can't see. When something goes wrong - a breach, an outage, a suspicious login - logs are how you figure out what happened. Without centralized logging, you're flying blind during the worst moments. **When:** Post-PMF, once you have paying customers. Before that, your cloud provider's built-in logging is enough. **First step:** Pick any centralized logging service (you probably already have one for ops) and make sure security-relevant events go there too.

**Common mistake:** Deploying a SIEM before you have someone to watch the alerts. A SIEM without an operator is an expensive log store with an inbox nobody reads.

- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) - What to log, what not to log, and how to log securely.
- [Wazuh](https://wazuh.com/) - Open-source security platform providing SIEM, XDR, and compliance capabilities.
- [Sigma Rules](https://github.com/SigmaHQ/sigma) - Generic signature format for SIEM systems. Write detection rules once, convert to any SIEM format.
- [Elastic Security](https://www.elastic.co/security) - SIEM and endpoint security built on the Elastic Stack. Free and open basic tier.
- [CrowdStrike Falcon Go](https://www.crowdstrike.com/en-us/pricing/falcon-go/) - Endpoint protection designed for small businesses. Simpler than enterprise offerings.
- [Detection Engineering Weekly](https://www.detectionengineering.net/) - Newsletter covering detection engineering concepts and tools.
- [Matano](https://github.com/matanolabs/matano) - Open-source serverless security lake platform for AWS.

## Supply Chain Security

**Why you'd care:** Your app is mostly other people's code. The average startup has hundreds of dependencies, and any one of them could be compromised. The [XZ Utils backdoor](https://en.wikipedia.org/wiki/XZ_Utils_backdoor) showed that even well-maintained, well-known projects can be targeted. **When:** Day 1 for dependency scanning (it's free and automatic). Advanced tools (SLSA, Sigstore, SBOMs) become relevant at 50+ employees or when enterprise customers ask about your software supply chain. **First step:** Enable Dependabot and use a lockfile. That's it for now.

- [SLSA Framework](https://slsa.dev/) - Supply-chain Levels for Software Artifacts. A framework for ensuring the integrity of software artifacts.
- [Sigstore](https://www.sigstore.dev/) - Keyless code signing and verification. Eliminates the complexity of traditional code signing.
- [in-toto](https://in-toto.io/) - Framework for securing the integrity of software supply chains with attestations for each build step.
- [Syft](https://github.com/anchore/syft) - CLI tool for generating SBOMs from container images and filesystems.
- [GUAC](https://guac.sh/) - Graph for Understanding Artifact Composition. Aggregates software security metadata into a queryable graph.
- [OpenSSF Scorecard](https://securityscorecards.dev/) - Automated tool that assesses open-source projects for security risks. Check your dependencies before adopting them.
- [Renovate](https://github.com/renovatebot/renovate) - Automated dependency update tool supporting 60+ package managers. More configurable than Dependabot.
- [NIST Secure Software Development Framework (SSDF)](https://csrc.nist.gov/Projects/ssdf) - Guidance on integrating security into your software development lifecycle.

## AI/ML Security

**Why you'd care:** If your product uses an LLM, prompt injection can expose customer data, bypass access controls, or make your product do things you didn't intend. If your LLM can call APIs, execute code, or access databases, a prompt injection is a potential full-system compromise. **When:** Before you ship any AI-powered feature. **First step:** Review the OWASP LLM Top 10 - it takes an hour and covers the attacks you're most likely to face.

**Most startups consume AI APIs (OpenAI, Anthropic, Google) rather than training models.** If that's you, your biggest risks are: leaking API keys (see [Secrets Detection](#secrets-detection)), sending customer PII to third-party models without a zero-data-retention agreement, and prompt injection if you expose LLM features to users. If you're *building* ML models or fine-tuning on customer data, the full section below applies.

- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/) - The definitive list of security risks for LLM-powered applications: prompt injection, data poisoning, and more.
- [OWASP Machine Learning Security Top 10](https://owasp.org/www-project-machine-learning-security-top-10/) - Security risks specific to machine learning systems beyond LLMs.
- [NIST AI Risk Management Framework](https://airc.nist.gov/Home) - NIST's framework for managing risks associated with AI systems.
- [Anthropic's Responsible Scaling Policy](https://www.anthropic.com/news/anthropics-responsible-scaling-policy) - Anthropic's approach to evaluating and mitigating risks from increasingly capable AI systems.
- [MITRE ATLAS](https://atlas.mitre.org/) - Adversarial Threat Landscape for AI Systems. A knowledge base of adversarial techniques targeting ML systems.
- [Garak](https://github.com/NVIDIA/garak) - LLM vulnerability scanner by NVIDIA. Tests for prompt injection, data leakage, and other LLM-specific vulnerabilities.
- [LLM Guard](https://github.com/protectai/llm-guard) - Security toolkit for LLM interactions. Scans inputs and outputs for prompt injection, PII leakage, toxicity, and more.
- [AI Verify](https://aiverifyfoundation.sg/) - Open-source AI governance testing framework from Singapore's AI Verify Foundation.

#### AI Coding Agents / Vibe-Coding Security

AI coding tools (Claude Code, Codex, Copilot) are now part of real engineering workflows. Treat them as powerful but untrusted team members:

- **Treat AI-generated code as untrusted until reviewed.** Don't blindly merge AI output — scan for secrets, vulnerable dependencies, and insecure defaults.
- **Sandbox agent execution.** Never give coding agents default access to production credentials, cloud accounts, or write access to sensitive repos.
- **Require approvals for dangerous operations.** Shell execution, git push, external network calls, and access to secrets should require human approval.
- **Assume prompt injection can arrive anywhere.** Issues, PRs, docs, pasted code, and retrieved context can all carry malicious instructions that agents may execute.

**Default rule:** AI tools can read code, but shell execution, git push, external network calls, and access to secrets require human approval.

## Books

Whether you're a founder trying to understand security or an engineer going deeper. Start with The Phoenix Project if you've never thought about security as a business enabler.

- [The Phoenix Project](https://itrevolution.com/product/the-phoenix-project/) by Gene Kim, Kevin Behr, George Spafford - A novel about IT, DevOps, and helping your business win. Makes the case for security as an enabler, not a blocker.
- [Building Secure and Reliable Systems](https://google.github.io/building-secure-and-reliable-systems/) by Google - **Free online.** Google's guide to designing systems that are both secure and reliable.
- [Threat Modeling: Designing for Security](https://shostack.org/books/threat-modeling-book) by Adam Shostack - The definitive guide to threat modeling. Shostack also offers free resources: the [Elevation of Privilege](https://shostack.org/games/elevation-of-privilege) card game for collaborative threat modeling sessions and the [Threat Modeling Manifesto](https://www.threatmodelingmanifesto.org/).
- [The Web Application Hacker's Handbook](https://portswigger.net/web-security/web-application-hackers-handbook) by Dafydd Stuttard, Marcus Pinto - Deep dive into web application security. Paired with PortSwigger's free Web Security Academy.
- [Security Engineering](https://www.cl.cam.ac.uk/~rja14/book.html) by Ross Anderson - **Free online.** Comprehensive text on security engineering principles.
- [Alice and Bob Learn Application Security](https://www.wiley.com/en-us/Alice+and+Bob+Learn+Application+Security-p-9781119687405) by Tanya Janca - Approachable guide to application security. Great for developers new to security.
- [Zero Trust Networks](https://www.oreilly.com/library/view/zero-trust-networks/9781491962183/) by Evan Gilman, Doug Barth - Practical guide to implementing zero trust architecture.
- [Cybersecurity for Small Networks](https://nostarch.com/cybersecurity-small-networks) by Seth Enoka - Right-sized for startups and small organizations.

## Videos & Talks

Free courses and conference recordings. The Web Security Academy is especially valuable - it's hands-on and covers real vulnerabilities your product might have.

- [PortSwigger Web Security Academy](https://portswigger.net/web-security) - Free, comprehensive web security training with interactive labs. The single best resource for learning to find and fix web vulnerabilities.
- [SANS Cyber Aces Online](https://www.sans.org/cyberaces/) - Free introductory courses in cybersecurity fundamentals from SANS.
- [Hack the Box Academy](https://academy.hackthebox.com/) - Hands-on cybersecurity training with free tier. Great for upskilling developers.
- [PentesterLab](https://pentesterlab.com/) - Hands-on web security exercises. Free tier with essential exercises.
- [DEF CON Talks on YouTube](https://www.youtube.com/@DEFCONConference) - All DEF CON conference talks, free.
- [BSides Conference Talks](https://www.youtube.com/@SecurityBSides) - BSides security conference recordings. Often more practical than larger conferences.
- [Black Hat Talks on YouTube](https://www.youtube.com/@BlackHatOfficialYT) - World-class security research presentations.
- [LiveOverflow](https://www.youtube.com/@LiveOverflow) - Excellent YouTube channel on security concepts, CTF walkthroughs, and vulnerability research.

## Newsletters & Blogs

Subscribe to one or two to stay current without drowning in noise. tl;dr sec is the single best security newsletter for busy people.

- [tl;dr sec](https://tldrsec.com/) - The best security newsletter, period. Weekly roundup of security tools, blog posts, talks, and job listings by Clint Gibler.
- [Secure Patterns](https://securepatterns.dev/) - Practical security engineering patterns and guides for developers.
- [Krebs on Security](https://krebsonsecurity.com/) - Brian Krebs' investigative cybersecurity journalism. Read this to understand the threat landscape without the hype.
- [Schneier on Security](https://www.schneier.com/) - Bruce Schneier's blog on security, technology, and society. Thoughtful analysis, not fear-mongering.
- [The Hacker News](https://thehackernews.com/) - Breaking cybersecurity news, vulnerability disclosures, and data breach reports.
- [Daniel Miessler's Unsupervised Learning](https://danielmiessler.com/) - Security and AI analysis. Good at connecting security concepts to business value.
- [CloudSecList](https://cloudseclist.com/) - Weekly cloud security newsletter covering AWS, GCP, Azure, and cloud-native security.
- [Risky Business](https://risky.biz/) - Security news podcast and newsletter. Good for staying current.
- [Trail of Bits Blog](https://blog.trailofbits.com/) - Research blog from one of the top security consultancies.
- [Google Project Zero Blog](https://googleprojectzero.blogspot.com/) - Vulnerability research from Google's elite vulnerability research team.

## Security Vendor Programs for Startups

Many security vendors offer startup programs with discounted or free access. This table covers pricing and programs — see the topical sections above for what each tool does and when to use it. Inclusion here doesn't imply endorsement. Pick the smallest tool that solves your current problem.

| Vendor | Category | Free Tier / Startup Program | Notes |
|--------|----------|----------------------------|-------|
| [Vanta](https://www.vanta.com/) | Compliance | Startup program (discounted) | SOC 2, ISO 27001, HIPAA |
| [Drata](https://drata.com/) | Compliance | Startup pricing available | SOC 2, ISO 27001, HIPAA, PCI DSS |
| [Secureframe](https://secureframe.com/) | Compliance | Startup-friendly pricing | Compliance + onboarding workflows |
| [Snyk](https://snyk.io/) | AppSec (SCA/SAST) | Free tier (limited projects); startup program | |
| [Semgrep](https://semgrep.dev/) | AppSec (SAST) | Free OSS CLI; cloud free tier for small teams | |
| [GitHub Secret Protection](https://github.com/features/security) | AppSec | Free for public repos; paid add-on for private | Requires Team or Enterprise plan |
| [AWS Security Hub](https://aws.amazon.com/security-hub/) | Cloud | Pay-per-check (affordable) | AWS only |
| [Cloudflare](https://www.cloudflare.com/) | Cloud/Network | Generous free tier (DDoS, WAF, Zero Trust) | |
| [Datadog Security](https://www.datadoghq.com/product/security-platform/) | SecOps | Startup program available | Integrates with observability stack |
| [1Password](https://1password.com/business/) | IAM | Business plan ($7.99/user/month at time of writing) | |
| [Bitwarden](https://bitwarden.com/) | IAM | Free sharing for 2 people; Teams at per-user pricing | |
| [JumpCloud](https://jumpcloud.com/) | IAM/MDM | Free trial available | |
| [Kolide](https://www.kolide.com/) | Device Trust | Pairs with Okta | |
| [HackerOne](https://www.hackerone.com/) | Bug Bounty | Platform fee varies | Largest bug bounty platform |
| [Bugcrowd](https://www.bugcrowd.com/) | Bug Bounty | Platform fee varies | Pen test + bug bounty + VDP |
| [Socket](https://socket.dev/) | AppSec (Supply Chain) | Free for OSS repos + limited private repos | Behavioral analysis, not just CVEs |
| [Aikido Security](https://www.aikido.dev/) | AppSec (All-in-one) | Free tier for small teams | SAST, SCA, DAST, cloud, secrets in one platform |
| [Doppler](https://www.doppler.com/) | Secrets Management | Free tier for small teams | Centralized secrets for production environments |
| [Infisical](https://infisical.com/) | Secrets Management | Free tier for small teams | Open-source secrets management platform |
| [Conveyor](https://www.conveyor.com/) | Security Questionnaires | Varies | Automates security questionnaire responses |
| [SafeBase](https://safebase.io/) | Trust Center / Questionnaires | Varies | Customer-facing security page + questionnaire automation |
| [Cobalt](https://www.cobalt.io/) | Pentesting | PtaaS model | Faster turnaround than traditional |

## Hiring & Building Security Teams

**Why you'd care:** At some point, security becomes an unavoidable distraction from scaling your business. Your engineers are spending time on security questionnaires instead of building features. Customers are asking questions nobody on your team can answer. That's the signal. **When:** Start thinking about it at 20 employees; hire between #20 and #40. **First step:** Read Rami McCarthy's rule of thumb below - it's the most nuanced take on timing.

**Common mistake:** Waiting until you're overwhelmed. Hiring takes months, security debt accumulates fast, and "we're drowning" is a less compelling pitch to candidates than "we're building proactively."

**The progression** (per [Thomas Ptacek](https://news.ycombinator.com/item?id=32667175)):
1. Sr. Security Engineer (first hire - hands-on-keyboard, not a CISO)
2. Second Sr. Security Engineer
3. Director of Security
4. Build out security engineering, ops, and IT security
5. Promote Director to VP; hire team managers
6. Get big
7. CISO (this is a "marquee resume" hire)

- [Starting Up Security - Hiring](https://scrty.io/management) - Ryan McGeehan's guide on timing, role definition, and what to look for in your first security hire.
- [The First Security Hire Rule of Thumb](https://ramimac.me/start-security) - Rami McCarthy's flexible rule of thumb for when to make your first security hire, with practical antipatterns to avoid and nuance beyond simple headcount ratios.
- [Your First 90 Days as a Founding Security Engineer](https://goauthentik.io/blog/2024-06-11-first-90-days/) - Practical playbook for what to do when you're the first and only security person at a 30-100 person company.
- [Startup Security: Ratios and a 24-Month Hiring Plan](https://theporkskewer.medium.com/startup-security-ratios-and-a-24-month-hiring-plan-83792b7491af) - 1:40 security-to-employee ratio benchmark and a quarterly hiring plan for scaling from zero to a multi-person team.
- [The Startup Guide to Your First Security Hire](https://www.vanta.com/resources/first-security-hire) - When, why, and how to make your first security hire.
- [How to Build a Startup Security Team](https://goteleport.com/blog/how-to-build-a-startup-security-team/) - Security experts on when to hire, build internal vs. external, and evaluating candidates (certifications are not the signal).
- [Security Job Descriptions by Ryan McGeehan](https://scrty.io/) - Open-source security job descriptions. Save time writing your first security role posting.
- [NCSC Board Toolkit](https://www.ncsc.gov.uk/collection/board-toolkit) - The UK NCSC's board toolkit for cyber security. Helps determine what security leadership you need and when.
- [NICE Framework Resource Center](https://www.nist.gov/itl/applied-cybersecurity/nice/nice-framework-resource-center) - NIST's framework for understanding cybersecurity roles, skills, and career paths.
- [Security Champions Program Guide](https://owasp.org/www-project-security-culture/stable/4-Security_Champions/) - OWASP's guide to building a security champions program. Scales security expertise across engineering teams without building a huge security team.

## Incident Response

**Why you'd care:** The difference between a minor event and a catastrophe is whether you had a plan before it happened. Without a plan, a breach turns into a week of panic, ad-hoc communication, and avoidable mistakes. With a plan, it's a structured process you've practiced. **When:** Write a plan once you have 10+ employees and real customers. Before that, "call the CEO" is your plan and that's fine. **First step:** Use the PagerDuty template below - it's free, battle-tested, and takes an afternoon.

- [NIST Computer Security Incident Handling Guide (SP 800-61 Rev 3)](https://csrc.nist.gov/pubs/sp/800/61/r3/final) - The authoritative guide to incident handling, now aligned with the NIST Cybersecurity Framework 2.0.
- [PagerDuty Incident Response Guide](https://response.pagerduty.com/) - **Free and open-source.** PagerDuty's battle-tested incident response documentation. Start here.
- [Incident Response Plan Template by Counteractive](https://github.com/counteractive/incident-response-plan-template) - Open-source, ready-to-customize incident response plan template.
- [FIRST (Forum of Incident Response and Security Teams)](https://www.first.org/) - Global forum for incident response teams. Resources, standards, and the FIRST CVSS calculator.
- [AWS Security Incident Response Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/welcome.html) - AWS-specific incident response guidance.
- [Atlassian Incident Management Handbook](https://www.atlassian.com/incident-management/handbook) - Practical handbook covering incident roles, communication templates, and postmortem processes.
- [GitLab Incident Response Guidance](https://handbook.gitlab.com/handbook/security/product-security/vulnerability-management/incident-response-guidance/) - GitLab's public incident response procedures. A real-world example of how a company handles security incidents.
- [Have I Been Pwned](https://haveibeenpwned.com/) - Free service for checking if email addresses appear in data breaches. Set up domain monitoring.

## Further Reading

Longer-form articles and essays worth reading when you have time. These shaped how the industry thinks about startup security.

- [Google's BeyondCorp Papers](https://cloud.google.com/beyondcorp) - The foundational papers on zero trust architecture from Google.
- [OWASP Proactive Controls](https://owasp.org/www-project-proactive-controls/) - The ten most important security techniques to build into every project.
- [Securing DevOps](https://www.manning.com/books/securing-devops) - Companion resources for the book by Julien Vehent. Practical DevSecOps guidance.
- [Zero Trust Architecture by NIST (SP 800-207)](https://csrc.nist.gov/publications/detail/sp/800-207/final) - NIST's authoritative guide to zero trust architecture concepts and deployment models.
- [Ask HN: IT Security Checklist for Startups?](https://news.ycombinator.com/item?id=32662385) - Rich practitioner discussion on startup security priorities. Key insights: Google SSO + Tailscale as the one-two punch, skip MDM until you have bandwidth, CloudTrail is non-negotiable on AWS, SOC 2 is documentation not reality, hire between employee #20-40, your first hire should be a Sr. Security Engineer not a CISO, and don't waste time on DNSSEC or DLP.

---

## Contributing

Contributions are welcome! Please read the [contribution guidelines](CONTRIBUTING.md) first.

## License

[![CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](LICENSE).
