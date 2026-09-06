# Secure Cloud Architecture

## Student Information

Name: (add your name)
Section: (add your section)
Course: (add your course)
Date: September 6, 2026

## Project Description

This activity demonstrates a proposed secure cloud architecture for a Student Management Application. The repository contains a simple web page (`index.html`) and a written security plan. It does not deploy real cloud infrastructure.

## Architecture

Users → CDN → Load Balancer → Application Servers → Private Database

- Users reach public edge services only.
- Application servers run in a private subnet.
- The database stays private and is reachable only from the application servers.

## Security Controls

- IAM
- MFA
- Firewall / Security Groups
- Private Subnets
- Encryption
- Logging
- Monitoring
- Backups

## Repository Files

- `index.html` — sample Student Management System page
- `security-plan.md` — architecture, public/private resources, security controls, least privilege, shared responsibility, and architecture questions
- `README.md` — project overview and student information
