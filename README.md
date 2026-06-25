# Ecommerce Platform

## Overview

Ecommerce Platform is a cloud-native microservices system built using:

- Java 17
- Spring Boot 4.5
- PostgreSQL
- AWS ECS Fargate
- SQS FIFO
- SNS
- Terraform

## Business Domains

- User Management
- Order Management
- Payment Processing
- Notification Delivery

## Services

| Service | Responsibility |
|----------|----------------|
| user-service | User lifecycle and authentication |
| order-service | Order lifecycle management |
| payment-service | Payment and refund processing |
| notification-service | Email and event notifications |

## Architecture

The system follows:

- Domain Driven Design
- Hexagonal Architecture
- Event-Driven Communication
- Database per Service

## Documentation

See:

- docs/business
- docs/architecture
- docs/aws
- docs/database
- docs/adr