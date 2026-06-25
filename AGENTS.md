# AI Agent Instructions

## Purpose

This repository contains the Ecommerce Platform microservices system.

Agents must understand business rules, architecture decisions, and operational constrains before implementing changes.

## Required Reading

Always read:
- README.md
- docs/business/*
- docs/architecture/*
- docs/aws/*
- docs/database/*
- docs/adr/*
before making implementation decisions.

## Engineering Principles
Follow:
- Domain Driven Design
- Hexagonal Architecture
- Database per Service
- Event-Driven Communication

## Development Workflow
Before implementation:
1. Analyze requirements
2. List assumptions
3. Propose implementation plan
4. Wait for approval
5. Implement incrementally
6. Generate tests
7. Perform security review

## Coding Constraints
Always:
- Use Java 17
- Constructor injection
- Immutable DTOs
- OpenAPI documentation
- Testcontainers for integration tests
Never:
- Use field injection
- Access another service database
- Put business logic inside controllers
- Use Lombok @Data on entities
