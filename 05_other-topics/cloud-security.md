---
title: Cloud Security
parent: Emerging Topics in Security
nav_order: 3
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Cloud Security

The way organizations design and run their infrastructure has undergone a profound transformation. What was once predominantly built on on-premises data centers with clear physical perimeters and direct hardware control has increasingly moved to the cloud. Recent reports shows that over half of enterprise now run in public clouds. Major organizations across industries have led this transition. Netflix operates almost entirely on AWS, while companies such as Capital One have closed multiple data centers and shifted the majority of their workloads to the cloud. Similar patterns can be seen at organizations like Airbnb, Spotify, and numerous financial institutions that now rely heavily on public cloud platforms for core operations.

Up to this point in the book, we have focused primarily on traditional on-premises networking and the security challenges that arise when data moves across networks you largely control. In the cloud, that clear perimeter largely disappears.

Your virtual machines, containers, databases, and storage live inside someone else’s data centers. Thousands of other customers share the same physical infrastructure. Networking is now _software-defined_: virtual private clouds (VPCs), security groups, and route tables replace traditional switches and firewalls. Most services are accessed through APIs rather than direct network connections.

In this environment, the same fundamental problems (lack of authentication at lower layers, best-effort delivery, and untrusted intermediaries) still exist. They have simply moved into a new, highly dynamic, and massively shared environment.

## How the Cloud Changes the Network Security Problem

The cloud fundamentally changes many of the assumptions you may have formed while studying traditional networking and on-premises infrastructure.

In a traditional on-premises setup, organizations typically maintained full physical control over their hardware, data centers, and facilities. They could establish a clearly defined network perimeter, usually protected by firewalls and other security devices at the edge. The entities that could potentially sit “on-path” were relatively limited: mainly ISPs, compromised internal devices, or physical intruders with access to the premises. Breaches were most commonly caused by weaknesses in network protocols or unpatched systems, and infrastructure changed at a relatively slow pace because of the time and effort required for hardware procurement and deployment. As a result, the primary attack surface centered around network protocols and the systems running on them.

Cloud environments operate under a very different model. The cloud provider now controls the underlying hardware, physical facilities, power systems, and core networking infrastructure. The traditional network perimeter becomes blurred or even disappears in many architectures, as most services and resources are accessed through APIs rather than direct network connections. Potential adversaries “on-path” can include not only the usual suspects, but also other tenants sharing the same infrastructure, misconfigured services, and even staff at the cloud provider. The dominant cause of security incidents in the cloud has shifted away from protocol-level exploits toward **misconfigurations**, particularly in identity and access management (IAM), storage permissions, and networking settings. Infrastructure can now be created, modified, and destroyed at an extremely rapid pace through infrastructure-as-code practices. Consequently, the attack surface expands well beyond network protocols to encompass APIs, identities, configurations, and the broader supply chain of cloud services and dependencies.

## Cloud Computing Fundamentals

This section provides the minimum background required to understand the security implications discussed later in the chapter. If you have already taken a cloud computing course, you might skim this section.

### Service Models: IaaS, PaaS, and SaaS

Cloud providers offer services along a spectrum of control and responsibility. The three main service models are:

| Service Model | Full Name                   | What the Provider Manages                     | What You Manage                         | Security Implication            |
| ------------- | --------------------------- | --------------------------------------------- | --------------------------------------- | ------------------------------- |
| **IaaS**      | Infrastructure as a Service | Physical hardware, virtualization, networking | OS, applications, data, configurations  | Highest customer responsibility |
| **PaaS**      | Platform as a Service       | Hardware, OS, runtime, middleware             | Applications, data, some configurations | Medium responsibility           |
| **SaaS**      | Software as a Service       | Everything (application + infrastructure)     | Only your data and access policies      | Lowest customer responsibility  |

As you move from IaaS to SaaS, the provider takes on more security responsibilities, but you lose flexibility and visibility into the underlying infrastructure. Most serious cloud breaches still occur because customers misconfigure the parts they _do_ control.

### Deployment Models

Cloud environments can be deployed in several ways:

- **Public Cloud**: Services run on shared infrastructure owned by the provider (e.g., AWS, Azure, GCP). This is the most common model for new projects.
- **Private Cloud**: Cloud technologies deployed within an organization’s own data centers.
- **Hybrid Cloud**: A combination of public and private clouds, often connected via secure links.
- **Multi-Cloud**: Using multiple public cloud providers simultaneously (for resilience, cost optimization, or regulatory reasons).

Most organizations today operate in _public_ or _hybrid_ environments. This means workloads often run alongside other customers on the same physical infrastructure.

### Key Cloud Abstractions

Cloud computing is built on several important abstractions that dramatically change how we think about networking and security:

- **Virtualization**: Physical servers are divided into many virtual machines (VMs) or containers.
- **Software-Defined Networking (SDN)**: Networking functions such as routing, firewalls, and load balancing are implemented in software rather than dedicated hardware.
- **API-Driven Control**: Almost everything is managed through APIs instead of physical consoles or SSH.
- **Ephemeral Resources**: Virtual machines, containers, and serverless functions can be created and destroyed in seconds.
- **Shared (Multi-Tenant) Infrastructure**: Compute, storage, and networking resources are shared among many customers.

These abstractions provide enormous flexibility and speed, but they also introduce new security challenges that did not exist in traditional on-premises data centers. The speed of change makes misconfigurations more likely and harder to track, while the API-first design turns identity and access control into the primary security boundary.

### The Shared Responsibility Model

The single most important concept in cloud security—and the one that causes the most confusion—is the **Shared Responsibility Model**.

In traditional on-premises environments, organizations were responsible for _everything_: physical facilities, hardware, operating systems, applications, data, and network configuration. Cloud providers fundamentally change this equation.

> **Shared Responsibility Model**: The cloud provider is responsible for the security _of_ the cloud. The customer is responsible for security _in_ the cloud.

This division allows providers to achieve massive economies of scale while giving customers flexibility. However, it also creates a dangerous gray area where responsibility can easily fall through the cracks.

The following table shows a generalized view of how responsibility is typically divided:

| Component                        | Cloud Provider Responsibility              | Customer Responsibility                                 |
| -------------------------------- | ------------------------------------------ | ------------------------------------------------------- |
| **Physical Infrastructure**      | Data centers, servers, networking hardware | None                                                    |
| **Hypervisors / Virtualization** | Management and isolation of VMs/containers | None                                                    |
| **Host Operating Systems**       | For managed services (e.g., Lambda, RDS)   | For IaaS VMs (patching, hardening, configuration)       |
| **Networking**                   | Global backbone, physical routers          | VPC configuration, security groups, routing rules       |
| **Storage**                      | Underlying storage durability              | Bucket permissions, encryption, access control          |
| **Identity & Access (IAM)**      | The IAM service itself                     | Creating users/roles, applying least-privilege policies |
| **Applications & Data**          | None                                       | Code, configurations, data classification, encryption   |
| **Compliance**                   | Provider certifications (SOC 2, ISO, etc.) | Customer-specific configurations and controls           |

**Note**: The exact division of responsibilities varies slightly between providers (AWS, Azure, GCP, etc.), but the core principle remains the same.

The amount of responsibility you carry depends heavily on the service model:

- **IaaS** (e.g., EC2, Azure VMs): You have the most responsibility, similar to traditional servers.
- **PaaS** (e.g., App Engine, Azure App Service): The provider manages the platform; you focus on applications and data.
- **SaaS** (e.g., Office 365, Salesforce): The provider manages almost everything; you primarily manage users, permissions, and data.

### Cloud Network Architecture and Security Controls

The networking concepts you studied earlier (IP routing, firewalls, and transport protocols) do not disappear in the cloud—they are virtualized and made programmable.

A **Virtual Private Cloud (VPC)** is your own isolated virtual network inside a cloud provider’s environment. Key components include:

- **Subnets**: Divisions of your VPC’s IP address space.
- **Route Tables**: Define how traffic is routed between subnets and to the internet.
- **Internet Gateway**: Allows resources in your VPC to communicate with the public internet.
- **NAT Gateways**: Allow resources in private subnets to reach the internet while remaining unreachable from outside.

From a security perspective, a VPC provides logical isolation from other tenants, but it is still software-defined and managed entirely through APIs.

#### Security Groups and Network ACLs

Cloud providers offer two primary mechanisms for controlling network traffic:

| Feature                  | Security Groups                               | Network ACLs                                   |
| ------------------------ | --------------------------------------------- | ---------------------------------------------- |
| **Stateful / Stateless** | Stateful (like a modern firewall)             | Stateless (like traditional packet filters)    |
| **Scope**                | Attached to individual instances or ENIs      | Applied at the subnet level                    |
| **Default Behavior**     | Deny all inbound, allow all outbound          | Allow all traffic by default                   |
| **Rule Evaluation**      | Only allow rules are evaluated                | Both allow and deny rules (processed in order) |
| **Use Case**             | Fine-grained control for individual resources | Broad subnet-level traffic filtering           |

**Security Groups** are the most commonly used control. They act like a distributed firewall that follows your virtual machines or containers wherever they go.

#### Software-Defined Networking Features

Cloud networking is highly programmable and includes powerful features such as:

- Load balancers
- Private endpoints / VPC endpoints (private access to services without traversing the public internet)
- VPC peering
- Transit Gateways

Each new abstraction adds flexibility but also expands the configuration surface that must be secured.

### Cloud Metadata Services — A Critical Attack Surface

One of the most important (and dangerous) features in cloud environments is the **Instance Metadata Service (IMDS)**.

Every virtual machine can query a special link-local address (`169.254.169.254`) to obtain information about itself, including temporary IAM credentials, user data, and network configuration.

**IMDSv1** (the older version) has no authentication requirement. An attacker who can trick a server into making an internal request (via Server-Side Request Forgery — SSRF) can steal IAM credentials and potentially take over the entire cloud account.

**IMDSv2** adds a session token requirement, significantly reducing this risk. Organizations should disable IMDSv1 wherever possible.

## 3. Identity and Access Management (IAM)

In traditional on-premises environments, security relied heavily on network perimeters defined by firewalls, IP ranges, and physical boundaries.
In the cloud, this model changes significantly. Because almost all interactions happen through APIs rather than direct network access, _identity becomes the primary control plane_.
Every action, e.g., launching a virtual machine, reading from storage, or calling an API, is authorized based on _who_ is making the request and _what permissions_ they have been granted.

As a result, Identity and Access Management (IAM) is one of the most critical security controls in any cloud environment.

### IAM in the Cloud

Cloud platforms are fundamentally API-driven.
There is no traditional “inside the network” versus “outside the network” distinction in the same way as on-premises setups.
Resources are accessible from anywhere on the internet as long as the caller presents valid credentials and has the necessary permissions.

This shift has important security implications:

- A compromised credential or overly permissive policy can give an attacker access from anywhere in the world, without needing to breach network defenses.
- Attackers often do not need to exploit protocol vulnerabilities. They can simply use legitimate-looking API calls with stolen or misused credentials.
- Misconfigurations in identity and access policies are among the most common causes of cloud breaches.

Because of this, strong IAM practices effectively become the new perimeter.

### Core IAM Concepts

Cloud providers offer several core building blocks for managing identity and access:

- **Users** represent individual people or applications that need access to resources.
- **Groups** allow you to organize users and apply permissions collectively.
- **Roles** provide temporary, assumable sets of permissions. They are especially useful for workloads and services because they avoid the need for long-lived credentials.
- **Policies** are JSON documents that explicitly define which actions are allowed or denied on specific resources.
- **Service accounts** (or service principals) serve as identities for applications, automation, and services rather than human users.

One of the most important security improvements in cloud IAM is the widespread use of **temporary credentials**.
Instead of creating permanent access keys, workloads assume roles that issue short-lived tokens. This significantly reduces the risk and impact if credentials are ever exposed.

### IAM Policies and Least Privilege

IAM policies are typically written in JSON and follow an explicit allow/deny model, where denies generally take precedence.

Imagine you have a storage bucket that contains sensitive customer data. Using IAM, you can define precise rules such as:

- Only members of the “Data Analysts” group can read files from the bucket.
- Only a specific automated service (using a service account) can write new files to the bucket.
- No one else, including other employees or external users, is allowed any access.

These rules are enforced through policies attached to users, groups, or roles. Even if an attacker somehow reaches the network level, they cannot access the data unless they also possess valid credentials that satisfy the IAM policies.

Here is a simple example of an AWS IAM policy that allows reading objects from a specific S3 bucket:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

The most effective way to use IAM is to follow the _principle of least privilege_, i.e., granting each identity only the minimum permissions required to perform its specific job.
Common mistakes that violate this principle include:

- Using overly broad wildcards (`*`) in policies
- Assigning high-privilege roles such as `AdministratorAccess` to non-administrative users or workloads
- Relying on long-lived access keys instead of temporary role-based credentials

### 5.4 Federation and Single Sign-On (SSO)

Most organizations do not manage cloud user accounts in isolation. Instead, they connect their cloud Identity and Access Management (IAM) system to a central **enterprise identity provider** using a technique called **federation**.
Through federation, users authenticate using their existing corporate credentials rather than creating separate usernames and passwords for the cloud.

When a user signs in, the enterprise identity provider verifies their identity and issues a trusted security token.
The cloud provider accepts this token and grants the user temporary access to cloud resources according to pre-defined permission mappings.
This model is commonly known as **Single Sign-On (SSO)**.

Consider an employee named Sarah who needs access to her company’s AWS environment. Instead of logging into AWS with a separate IAM user account and password, Sarah goes to the AWS console and selects “Sign in with SSO”. She is redirected to her company’s Okta login page (the organization’s centralized identity provider), where she enters her corporate username and password (and completes MFA).

Once Okta successfully authenticates her, it sends a secure token to AWS. AWS trusts this token because federation has already been configured between Okta and AWS. Based on Sarah’s group membership in Okta (for example, the “Engineering” group), AWS automatically grants her the appropriate permissions such as read access to certain S3 buckets and the ability to start EC2 instances in the development account.

From Sarah’s perspective, she only needs to remember one set of corporate credentials. From the organization’s perspective, access is centrally controlled: when Sarah changes teams or leaves the company, her permissions can be updated or revoked in one place (the enterprise identity provider) rather than in every cloud account.

### 5.5 Common IAM Attacks and Misconfigurations

IAM-related issues are among the leading causes of cloud security breaches. Attackers frequently exploit overly permissive roles that allow them to escalate privileges far beyond what was originally intended. Another common problem is the exposure of long-lived access keys in code repositories, configuration files, or public logs, which gives attackers immediate and persistent access to cloud resources.

Publicly accessible instance metadata services (particularly the older IMDSv1) can also leak temporary credentials to anyone who can reach the metadata endpoint. In addition, attackers exploit “confused deputy” scenarios, where one legitimate service is tricked into performing actions using the permissions of another service.

Once an attacker obtains valid IAM credentials (whether through any of the methods above), they can often move laterally across the entire cloud account with very little traditional network activity. This makes IAM misconfigurations especially dangerous in cloud environments.

### 5.6 IAM Security Best Practices

Effective identity and access management is one of the most important layers of defense in cloud environments. Because the cloud is heavily API-driven and permissions are often the primary attack vector, organizations should follow several core practices to reduce risk.

1. Whenever possible, use **IAM roles with temporary credentials** rather than long-lived access keys.
   Temporary credentials automatically expire, which significantly reduces the impact if they are accidentally exposed or stolen.
   Long-lived keys should be avoided for most workloads and only used in limited, well-controlled cases.

2. A foundational principle is to **enforce least privilege** and regularly review permissions.
   Grant users, roles, and services only the minimum permissions required to perform their tasks.
   Permissions should be audited periodically, as requirements change over time and overly broad access can accumulate.

3. For all human user accounts, **enable multi-factor authentication (MFA)**.
   MFA adds a critical second factor of authentication and greatly reduces the risk of account compromise even if a password is stolen or guessed.

4. IAM policies should take advantage of **policy conditions** to add extra layers of protection.
   Conditions can restrict access based on factors such as source IP address, whether MFA has been used, or specific time windows.
   These conditions make it harder for attackers to abuse stolen credentials from unexpected locations or at unexpected times.

5. Access keys that must be used should be **rotated and monitored aggressively**.
   Regular rotation limits the window of opportunity if a key is compromised, while monitoring helps detect unusual usage patterns that may indicate a breach.

6. Finally, organizations should use tools such as **IAM Access Analyzer** (or equivalent services from other providers) to automatically detect overly permissive policies and resources that are accessible from outside the intended scope.
   These tools provide continuous visibility into potential misconfigurations that might otherwise go unnoticed.

## Data Protection in the Cloud

Cryptography remains one of the most important tools in your security toolkit when moving to the cloud.
Even with strong network controls and IAM, data must still be protected both while stored and while moving between services.
Cloud providers make strong encryption widely available, but responsibility for using it correctly remains with the customer.

**Encryption in transit** protects data as it moves between clients, services, and across the network.
All major cloud services support TLS for this purpose.
Providers typically manage the certificates themselves and enforce TLS 1.2 or higher by default.
As a customer, your main responsibilities are to avoid unencrypted protocols such as plain HTTP or FTP, ensure that your applications properly validate certificates, and use private endpoints or VPC endpoints whenever possible so that traffic does not need to traverse the public internet.

**Encryption at rest** protects data while it is stored on disk or in object storage.
Cloud providers offer several convenient options to achieve this.
Most storage services enable default server-side encryption (SSE) automatically.
For greater control, you can use customer-managed encryption keys (CMEK), which allow you to manage the keys in a key management service.
In higher-security scenarios, you can also perform client-side encryption, meaning you encrypt the data yourself before uploading it to the cloud.
This approach gives you full control over the encryption process but requires more operational effort on your part.

## Key Management

Effective key management is one of the most important aspects of encryption in the cloud.
Even strong encryption provides little protection if the keys are poorly managed or easily accessible to attackers.
Most cloud providers offer a dedicated **Key Management Service (KMS)** that gives customers several options for handling encryption keys.

The simplest option is **provider-managed keys**. In this model, the cloud provider creates, stores, and manages the keys on your behalf.
This approach is convenient for general-purpose data and requires minimal operational effort, but it gives you less visibility and control over the keys.

For more sensitive or regulated workloads, **customer-managed keys (CMK)** are usually preferred.
With CMKs, you create and manage the keys yourself through the provider’s KMS.
This gives you greater control over key lifecycle, access policies, and auditing, though it also increases operational responsibility and cost.

For the highest security and compliance requirements, organizations can use **Hardware Security Modules (HSMs)**.
These dedicated hardware devices provide strong physical and logical protection for keys.
Cloud providers offer managed HSM services that combine the security benefits of hardware with the scalability of the cloud, though this option is the most complex and expensive to operate.

As a general rule, use customer-managed keys for any sensitive or regulated data rather than relying solely on provider-managed keys.
In addition, rotate encryption keys on a regular schedule and maintain detailed audit logs of key usage.
Regular rotation limits the potential damage if a key is ever compromised, while auditing helps detect unauthorized access or unusual activity.

| Option                          | Managed By      | Use Case                         | Security Trade-off                    |
| ------------------------------- | --------------- | -------------------------------- | ------------------------------------- |
| Provider-managed keys           | Cloud Provider  | General purpose data             | Simpler, less control                 |
| Customer-managed keys (CMK)     | You             | Regulated data, high sensitivity | More control, higher operational cost |
| Hardware Security Modules (HSM) | You (via cloud) | Highest compliance requirements  | Strongest protection, most complex    |

## Common Attack Vectors and Real-World Consequences

Even when organizations implement strong foundational controls, cloud environments continue to suffer major security breaches. The majority of these incidents are not caused by sophisticated zero-day exploits in the cloud provider’s infrastructure. Instead, they stem from **customer misconfigurations** and poor security hygiene on the customer side.

Several attack techniques have become especially common and damaging in cloud environments.

One of the most frequent is **misconfigured object storage**, where storage buckets or containers are left publicly readable or protected by weak access policies.
This often leads to large-scale data leaks containing personal information, credentials, or internal documents.

Another major vector is **IAM role abuse and privilege escalation**, in which an attacker obtains or assumes a role that has been granted excessive permissions, potentially resulting in full compromise of an entire cloud account.

A particularly dangerous technique involves **Server-Side Request Forgery (SSRF) attacks against the instance metadata service**.
By targeting the well-known metadata endpoint (typically `169.254.169.254`), attackers can steal temporary IAM credentials that applications running on cloud instances use to access other resources.

**Credential exposure** is also extremely common. Access keys are frequently left in source code, Git repositories, log files, or configuration files, giving attackers persistent access to cloud resources.

Finally, **supply chain attacks** have grown in importance. Attackers compromise container images, serverless function layers, or infrastructure-as-code templates, allowing them to insert backdoors directly into production workloads.

### Real-World Incidents

- **Uber (2017)**: Attackers gained access to Uber’s private GitHub repository and extracted the company’s AWS credentials. While the technical breach itself was serious, the incident became particularly infamous because Uber chose to pay the hackers $100,000 to keep it quiet rather than notifying the approximately 57 million affected customers and drivers.

- **Numerous S3 Bucket Exposures (2017 onward)**: Companies such as Dow Jones, Verizon, Accenture, and multiple government agencies exposed hundreds of millions of records through publicly accessible S3 buckets. In several cases, sensitive data remained publicly available for months or even years before being discovered.

- **Capital One (2019)**: A misconfigured Web Application Firewall (WAF) combined with an overly permissive IAM role allowed an attacker to access data from over 100 million customers through a Server-Side Request Forgery attack against the metadata service.

- **Toyota (2023)**: Misconfigured cloud storage exposed sensitive customer and vehicle data (including location information and personal details) belonging to approximately 2.15 million users in Japan. The data remained accessible for an extended period due to overly permissive cloud settings.

- **ShinyHunters / Nemesis Credential Theft (2024)**: Attackers exploited misconfigurations in public websites to steal thousands of AWS credentials and other secrets. They stored over 2 TB of stolen data in an open S3 bucket, which was later discovered by researchers. The operation remained active for several months.

These types of attacks continue to succeed at a high rate for several interconnected reasons:

- Cloud environments change rapidly (infrastructure as code, auto-scaling).
- Developers are often given broad permissions to “make things work”.
- Visibility into configurations across hundreds of accounts and services is difficult.
- The Shared Responsibility Model is frequently misunderstood.

## Modern Defenses and Best Practices

Cloud environments require a different security mindset than traditional on-premises infrastructure. Because infrastructure is dynamic, API-driven, and largely managed by the provider, security controls must focus heavily on **configuration**, **identity**, and **automation** rather than only on network perimeters or host hardening.

### Core Security Principles

Effective cloud security is built on a few timeless but critically important principles:

- **Least Privilege**: Grant users, roles, and services only the minimum permissions they need to do their job. This is the single most effective control against both accidental damage and malicious compromise.
- **Defense in Depth**: Never rely on a single control. Combine identity controls, network restrictions, encryption, logging, and monitoring so that the failure of one layer does not lead to total compromise.
- **Assume Breach**: Design systems and processes with the expectation that attackers will eventually gain some level of access. Focus on rapid detection, containment, and response.
- **Automation and Immutability**: Use Infrastructure as Code (IaC) and automated security checks in CI/CD pipelines. Manual configuration changes are a major source of errors and drift.

### Key Areas of Cloud Defense

#### 1. Identity and Access Management (IAM)

Identity is the new perimeter in the cloud. Strong IAM practices include:

- Prefer **IAM roles with temporary credentials** over long-lived access keys whenever possible.
- Enforce **least privilege** and conduct regular access reviews and permission audits.
- Enable **Multi-Factor Authentication (MFA)** for all human users and privileged accounts.
- Use **policy conditions** (source IP, MFA requirement, time restrictions, etc.) to add extra layers of protection.
- Implement tools such as IAM Access Analyzer to automatically detect overly permissive policies and unintended public access.

#### 2. Network and Infrastructure Hardening

Even though the traditional network perimeter is less relevant, network controls remain important:

- Use **private endpoints** and VPCs instead of exposing resources to the public internet.
- Apply strict **security groups and network ACLs** and follow the principle of least privilege for network access.
- Disable or restrict metadata service access where not needed, and protect against SSRF attacks targeting instance metadata.
- Segment workloads using separate accounts, VPCs, or subnets based on sensitivity and function.

#### 3. Data Protection

Protecting data remains a core responsibility:

- Encrypt sensitive data **in transit** (using TLS) and **at rest** (using server-side or customer-managed encryption keys).
- For highly sensitive workloads, consider **customer-managed keys (CMK)** or hardware security modules (HSMs) instead of provider-managed keys.
- Apply strict access controls and monitoring on storage services (object storage, databases, file systems).
- Classify data and apply appropriate protection levels based on sensitivity.

#### 4. Logging, Monitoring, and Threat Detection

You cannot secure what you cannot see:

- Enable comprehensive logging for all services (especially IAM, network, and data access events).
- Centralize logs and use automated analysis or security information and event management (SIEM) tools to detect anomalies.
- Implement **Cloud Security Posture Management (CSPM)** tools to continuously scan for misconfigurations.
- Set up alerting for high-risk activities such as permission changes, unusual login locations, or large data transfers.

#### 5. Automation and Secure Development

Security must be integrated into how cloud resources are created and changed:

- Use **Infrastructure as Code** with security scanning integrated into pipelines (shift-left security).
- Implement automated policy-as-code checks to prevent non-compliant resources from being deployed.
- Regularly scan container images, serverless functions, and dependencies for vulnerabilities.
- Automate response to common incidents where possible (e.g., automatically isolating compromised instances).
