# Secure Cloud Architecture Plan

This plan describes a proposed cloud design for a Student Management Application. The application lets authorized users view student information. The design is documented in GitHub only. It is not deployed on AWS, Azure, or Google Cloud.

Proposed request flow:

Users → CDN → Load Balancer → Application Servers → Private Database

```
Users
  |
  v
CDN (public)
  |
  v
Load Balancer (public)
  |
  v
Application Server 1 / Application Server 2 (private subnet)
  |
  v
Private Database (private subnet)
```

## CDN

The CDN (Content Delivery Network) stores cached copies of static content closer to users. Static files such as HTML, CSS, and images can be served from the CDN so pages load faster and origin servers receive fewer requests. The CDN is a public-facing entry point and should only cache content that is safe to expose.

## Load Balancer

The load balancer receives incoming HTTPS requests and distributes them across multiple application servers. It hides individual server addresses from the Internet, supports health checks, and helps the application stay available if one server fails. Users should reach the application through the load balancer, not by connecting to a server directly.

## Application Servers

Application servers process requests from users, such as showing student records after login. These servers should be placed in a private subnet. They should accept traffic only from the load balancer, and they should be the only resources allowed to talk to the database.

## Database

The database stores student records (name, student number, course, year level, and email). It must remain private and must not be directly accessible from the Internet. Only the application servers should be allowed to connect to it.

---

# Public and Private Resources

| Resource | Public or Private? | Explanation |
| --- | --- | --- |
| CDN | Public | Users on the Internet need to reach cached static content quickly. The CDN is designed as a public edge service. |
| Load Balancer | Public | The load balancer is the public entry point for application requests. It accepts HTTPS from the Internet and forwards traffic to private servers. |
| Application Server | Private | Application servers hold application logic and should sit in a private subnet. Only the load balancer should send traffic to them. |
| Database | Private | Student records are sensitive. The database must stay in a private subnet with no public IP and no direct Internet access. |

---

# Security Controls

## IAM

Access to the cloud environment should be limited to people who need it for their job.

- Administrators: full environment management (accounts, networks, security rules, backups).
- Developers: permission to update application code and view application logs, but not full account control.
- Instructors: permission to view student records through the application, not through the cloud console.
- Students: permission to view only their own information through the application.

Cloud console access should use unique user accounts. Shared passwords and unused accounts should not be allowed.

## MFA

Multi-Factor Authentication should be required for:

- Administrator accounts
- Developer accounts with access to the cloud console or source code
- Any account that can change IAM, networking, or database settings

MFA reduces the risk that a stolen password is enough to take over the environment.

## Firewall / Security Group

Allowed and blocked connections:

- Internet → Load Balancer = Allowed (HTTPS only)
- Load Balancer → Application Server = Allowed
- Application Server → Database = Allowed
- Internet → Application Server = Blocked
- Internet → Database = Blocked
- Database → Internet = Blocked except required updates through a controlled path

Security groups should allow the minimum ports needed (typically 443 to the load balancer, application ports from the load balancer, and the database port from application servers only).

## Encryption

Student information should be encrypted because it includes personal data such as names, student numbers, and email addresses.

- Encrypt data in transit with HTTPS/TLS between users, the load balancer, and application servers, and with encrypted connections to the database.
- Encrypt data at rest on the database and on backups.

Encryption helps protect records if traffic is intercepted or if a storage volume or backup file is stolen.

## Logging

These activities should be recorded:

- Successful and failed login attempts, Who viewed or changed student records, Changes to IAM users, roles, and permissions, Firewall / security group changes, Database connection attempts, and Backup success and failure events. Logs should be stored in a protected location that regular application users cannot modify.

## Monitoring

Suspicious activity that should be monitored includes:

- Repeated failed logins, Access attempts to the database from the Internet, Unusual traffic volume against the load balancer, Privilege changes outside normal hours, Application servers becoming unhealthy, and Backup jobs failing. Alerts should notify administrators so they can respond quickly.

## Backup

The database should have backups so student records can be restored after accidental deletion, ransomware, software bugs, or hardware failure. Backups should be encrypted, stored separately from the primary database, and tested with periodic restore drills.

---

# Principle of Least Privilege

Each role receives only the access needed for that role. Administrator access is not given to everyone.

| User | Allowed Access |
| --- | --- |
| Administrator | Manage cloud accounts, IAM, networking, security groups, application servers, database settings, logging, monitoring, and backups. Can restore data. Does not use the student app as a normal student. |
| Instructor | Log in to the application to view student records needed for teaching (class lists and academic details). Cannot access the cloud console, change IAM, or connect to the database directly. |
| Student | Log in to the application to view their own student information. Cannot view other students' private records, cannot access servers, and cannot access the database. |
| Developer | Update application code, view application logs, and test the application in a non-production or limited environment. Cannot change production IAM, cannot open the database to the Internet, and cannot grant administrator rights. |

---

# Shared Responsibility Model

| Responsibility | Cloud Provider or Customer? |
| --- | --- |
| Physical data center | Cloud Provider |
| Physical servers | Cloud Provider |
| User accounts | Customer |
| Student data | Customer |
| IAM permissions | Customer |
| Application security | Customer |
| Database access rules | Customer |
| Backups | Shared (provider may offer backup tools; the customer must enable, configure, protect, and test backups of student data) |

1. ## What does Security OF the Cloud mean?

Security of the cloud is the provider's responsibility. The provider protects the physical buildings, power, hardware, hypervisors, and the global infrastructure that runs cloud services.

2. ## What does Security IN the Cloud mean?

Security in the cloud is the customer's responsibility. The customer must configure accounts, IAM, networks, firewalls, encryption, the application, database access, and the protection of student data.

---

# Architecture Questions

3. ## Which resource should be directly accessible from the Internet?

The CDN and the load balancer should be directly accessible from the Internet. Application servers and the database should not.

4. ## Why should the database remain private?

The database holds student personal information. Keeping it private reduces the chance that attackers on the Internet can reach it, scan it, or steal records.

5. ## Why should users not connect directly to the database?

Users should only use the application. Direct database access bypasses login checks, input validation, and logging in the application. It also makes it easier to expose or change records by mistake.

6. ## What is the purpose of a load balancer?

A load balancer spreads incoming requests across multiple application servers, performs health checks, and provides a single public entry point so users do not connect to individual servers.

7. ## What happens if one application server fails?

The load balancer stops sending traffic to the failed server and continues sending requests to healthy servers. The application can remain available instead of going fully offline.

8. ## What is the purpose of a CDN?

A CDN caches static content close to users so pages load faster and origin servers handle less traffic.

9. ## Why should administrator accounts use MFA?

Administrator accounts can change the whole environment. MFA adds a second proof of identity so a stolen password alone is not enough to take over the account.

10. ## Why should administrator access not be given to every employee?

Administrator access is high risk. If every employee has it, a single compromised or careless account can delete data, open the database to the Internet, or disable security controls. Least privilege limits damage.

11. ## Why are logging and monitoring important?

Logging creates a record of who did what. Monitoring uses those records and system health signals to detect attacks, failures, and misconfigurations so administrators can respond before a small issue becomes a major breach.

12. ## Why are backups important?

Backups make it possible to recover student records after deletion, corruption, ransomware, or infrastructure failure. Without backups, lost data may be unrecoverable.
