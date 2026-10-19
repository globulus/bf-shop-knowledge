# Shopfront — overview

*Owned by the maintainer. Agents may propose changes in PRs.*

## Vision

Shopfront is a web shop for coffee gear: grinders, brewers, beans and filters. Customers
browse the catalog and place orders on the web or in the mobile app. The product exists
and is in use; new work changes it, it doesn't start it.

## Client

A stand-in for a client with an existing multi-repo product (Globuli's brownfield
testbed). Product issues come from Linear (team GTB), designs from Figma.

## Users

- **Customers** on the web shop and in the mobile app (Android and iOS). There is no
  login yet: the web is the customer `web-customer`, the app `demo-customer`.
- **The shop's staff** change the catalog and prices directly in the database for now.

## Goals

- Keep the web shop and the app working while the issues in Linear are built.
- Change the APIs and their clients together, in one release, when an issue needs it.

## Non-goals

- Payments, accounts and login, until an issue asks for them.
- Rewriting what works: the repos keep their build tools, frameworks and conventions.

## Modules

| Module | Purpose |
|---|---|
| catalog | Products, categories and prices (catalog-service) |
| orders | Placing, listing and cancelling orders (orders-service) |
| web | The web shop (React) |
| mobile | The app for Android and iOS (Flutter) |

## Constraints

- Five repos: orders-service (Spring Boot, Gradle), catalog-service (Spring Boot,
  Maven), web (React, npm), mobile (Flutter) and dev-env (Docker Compose for the whole
  product).
- The services publish OpenAPI specs; web and mobile keep copies in `openapi/` and
  generate their clients from them. An API change ships with its clients.
- Each repo has its own known debts (see their READMEs); fix one only when an issue
  asks for it.
