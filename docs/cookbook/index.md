# Cookbook

Task‑oriented guides you can copy and adapt.

## Infinite scroll with pagination

- Use BondFire to fetch pages and `XListResponse.merge` to combine
- Trigger next page on scroll threshold

## Optimistic updates

- Update UI first, send request, rollback on error
- Cache updated list, then reconcile with server

## File upload with progress

- Use Dio upload and emit progress to UI
- Retry on transient failures

## Deep links to feature routes

- Central parser, map to feature routes
- Handle notification taps by delegating to the same parser

## Force update dialog

- Remote config flag
- Modal gate at startup

## Feature flags

- Store flags in cache and refresh periodically
- Wrap UI/logic with simple gates
