---
name: api-contract-workflow
description: Review, design, or implement API changes across a client and server. Use for HTTP or streaming endpoints, request/response models, validation, error mapping, and synchronized client contracts.
---

Copyright 2026 domgao. Licensed under the Apache License, Version 2.0. See the repository LICENSE and NOTICE files.

# API Contract Workflow

Use this skill when an API task spans—or may affect—both a client and a server. Adapt the steps to the repository's language, framework, architecture, and established conventions; do not impose a specific stack.

## Establish scope and current behavior

1. Read repository instructions for the affected server and client areas. Check the working tree and preserve unrelated edits.
2. Respect task intent: audits and design requests produce findings or a proposal without edits; implementation requests update the code and affected contract.
3. Trace the actual path for each endpoint instead of inferring behavior from names or documentation:

   `UI action → client API function and type → route/endpoint → request model → handler/controller → application use case → response mapping → client consumer`

4. Follow domain, persistence, or infrastructure code only as far as needed to establish ownership, validation, error behavior, and response data.

## Keep responsibilities and data explicit

- The HTTP/SSE handler binds protocol data, validates boundary constraints, obtains trusted identity from the server-side auth context, calls the application layer, and maps the result to a protocol response. Keep business rules and repository access in their owners.
- Represent stable request and response contracts with named models and concrete field types. Avoid `Map<String, Object>`, generic `Object`, or unstructured strings as substitutes for known schemas. Use open-ended maps only when the domain truly allows arbitrary keys and the contract documents that shape.
- Name wire fields for their business meaning, not their implementation. Use the same name for the same concept across server DTOs, JSON, and client types; make units and distinctions explicit where useful (for example, `durationMs`, `sizeBytes`, `pageNo`, and `versionUuid`). Prefer plural names for collections and clear predicates such as `hasNext` for booleans. Avoid vague protocol names such as `data`, `value`, `result`, or `options` when a domain-specific name is available.
- Put request conversion beside its request model when that model owns the mapping (for example, `toCommand()`). Put response conversion on the target response model (for example, `Response.from(result)`), rather than in an unrelated outer utility class.
- Make list and nested object shapes explicit. If client state uses a different shape for editing convenience, convert it at the API boundary and keep wire types aligned with the server.
- Normalize and defensively copy response collections where needed. Keep comments about constructors concise and focused on behavior; do not repeat record/property field documentation in constructor comments.
- Keep server-side comments accurate and useful: endpoint docs explain purpose and meaningful parameter/response semantics; model docs explain field meaning, units, format, and nullable behavior when that is not evident from the type. Record component docs should describe each field once. Avoid comments that merely restate names or validation annotations, and remove stale comments when behavior changes.

## Name routes, HTTP operations, and handler methods consistently

- Read the existing route conventions before adding an endpoint. Treat the controller/base route as the API resource area, then group related routes by a clear resource or capability. In the Knowledge API example, `/api/knowledge/bases` groups knowledge-base operations, while `/api/knowledge/retrievals` groups retrieval operations.
- Choose the HTTP method for the operation's effect: `GET` reads without changing state; `POST` creates a resource or submits an operation that is not a simple resource replacement (including searches with a structured body and streaming requests); `PUT` updates a resource or its defined configuration; `PATCH` applies a partial update when the API supports it; `DELETE` removes or discards a resource. Do not choose a method only to match a Java method name.
- Keep the route's operation explicit and stable. The Knowledge API example uses camelCase action segments such as `/api/knowledge/bases/listKnowledgeBases`, `/api/knowledge/bases/createKnowledgeBase`, and `/api/knowledge/retrievals/retrieveEvidence`; follow the repository's established wire format for existing APIs instead of silently renaming routes. For a new API, use the project's documented route style consistently, and do not mix naming styles within the same route family.
- Include a resource identifier in the path when the operation targets one specific resource (as in `/api/knowledge/bases/{knowledgeBaseUuid}/updateDefaultStrategy`). Use query parameters for bounded filters or paging on reads, and a named request model in the body for structured commands or multi-resource input. Avoid encoding identifiers or complex request data into an opaque route or generic map.
- Name handler/controller methods with a precise action and domain object, and keep the client API function semantically aligned with that operation. Knowledge API examples include `listKnowledgeBases`, `createKnowledgeBase`, `updateDefaultStrategy`, `deleteKnowledgeBase`, `retrieveEvidence`, and `streamKnowledgeAnswer`. Avoid generic names such as `handleRequest`, `process`, or `doAction`; distinguish similar operations (for example, list document summaries versus list document versions).
- Ensure route action, HTTP method, handler method, client function, and API documentation describe the same operation. When an established route is awkward, preserve it unless a requested contract change includes the server route, all client callers, tests, and compatibility implications.
- Add or update route tests that assert the HTTP method and complete path, and check client call sites when changing either. A Java handler rename alone does not change the HTTP contract; a route or HTTP-method change does.

## Preserve identity and error semantics

- Treat client identifiers as resource selectors, never proof of ownership or authorization. Derive the current principal from the server-side security context and enforce access in the appropriate application/domain path.
- Preserve established status codes, error envelopes, and stable error codes unless the user requests a contract change.
- Translate boundary failures, including checked IO errors, into the established application/API error model. Do not expose raw exceptions or internal details to clients.
- Trace success, validation failure, authorization failure, not-found, conflict, and dependency failure paths when changing an endpoint.

## Handle streaming as a protocol

For SSE or similar streams, model payloads for each stable event shape with named types. Keep event names and JSON properties synchronized with the client. Trace startup, progress/evidence/delta events, terminal success and error events, cancellation, disconnect, timeout, and cleanup before changing stream behavior.

## Synchronize the client

When a request, response, or event shape changes:

1. Update the client-side API types, including required/optional fields and nullability.
2. Update serialization/deserialization, UI consumers, and any local-to-wire conversion.
3. Update the relevant API, component, or integration tests on both sides.
4. Compare field names, meanings, optionality, units, and JSON shapes directly across server models, client types, and tests. Fix naming drift at the contract boundary and update every caller that depends on the old name.

Preserve paths, field names, and error behavior unless a contract change is requested. When a change is necessary, identify affected callers and explain compatibility or persisted-data implications.

## Verify and report

- Read the repository's verification guidance and run the smallest credible server and client checks for the affected behavior. Expand validation for cross-module or high-risk changes.
- Distinguish unit/behavior tests from compilation, type checking, browser testing, and live-service verification. Report only commands actually run and their results.
- Review the final diff for unrelated changes, contract drift, ambiguous payloads, and formatting errors. Report changed endpoints/models, client updates, verification, and remaining limitations.
