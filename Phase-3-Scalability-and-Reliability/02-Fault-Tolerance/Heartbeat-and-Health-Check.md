Heartbeat and Health Check
===

# Overview
Heartbeat and health checks are monitoring mechanisms used to detect failures and assess system health. Heartbeats are periodic signals indicating a component is alive, while health checks actively probe components to verify they're functioning correctly. Both are critical for implementing automatic failover and maintaining high availability.

# Heartbeat Mechanisms

## What is a Heartbeat?
A heartbeat is a periodic signal sent by a component to indicate it's operational. If heartbeats stop, the system assumes the component has failed.

## Heartbeat Patterns

**Push-Based Heartbeat:** Components actively send "I'm alive" signals to a monitoring service at regular intervals. Simple to implement and low overhead, but can create network congestion with many components.

**Pull-Based Heartbeat:** Monitoring service actively queries components for their status. Centralizes monitoring logic and easier to adjust check frequency, but monitoring service becomes potential bottleneck.
