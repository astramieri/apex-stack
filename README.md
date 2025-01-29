# Full APEX Stack on Oracle Cloud Free Tier

This repository contains notes and configurations for setting up a full Oracle APEX stack on **Oracle Cloud Infrastructure (OCI)** using the Free Tier. 

The goal is to create a production-ready architecture that leverages OCI's free resources while maintaining scalability, security, and best practices.

## Basic Setup

See notes [here](./docs/basic_setup.md).

## Scenario 1

The architecture includes:
- **Oracle Autonomous Database (ADB)** with Oracle APEX enabled
- **NGINX** as a reverse proxy for APEX
- **Custom Domain** & SSL/TLS configuration for secure access

See notes [here](./docs/scenario_1.md).

## Scenario 2

The architecture includes:
- **Oracle Autonomous Database (ADB)** with Oracle APEX enabled
- **Oracle REST Data Services (ORDS)** for exposing APEX applications and APIs
- **NGINX** as a reverse proxy for ORDS and APEX
- **Custom Domain** & SSL/TLS configuration for secure access

See notes [here](./docs/scenario_2.md).

