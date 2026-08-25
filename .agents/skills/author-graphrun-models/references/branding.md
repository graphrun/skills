<!-- Centralizes public naming and stable compatibility identifiers for the skill package. -->
# Branding and compatibility

The public product name is `graphRun`, with a lowercase `graph` and uppercase
`R`. Use it in the README, skill description and title, and agent display
metadata. Keep deeper operating guidance product-neutral by referring to “the
MCP”, “the server”, or “the product”; this limits future renaming to the public
entry points and this file.

Do not rename compatibility-sensitive identifiers during a public rebrand:

- schema and contract names such as `mockflow.mcp.*`;
- MCP resource URIs beginning with `mockflow://`;
- opaque references and local placeholders such as `mfref2.*` and
  `mcp-local-*`;
- package, environment, header, extension, and media-type identifiers that
  intentionally retain `mockflow`.

The repository locator is `graphrun/skills`, and the current skill ID and
directory are `author-graphrun-models`. Unlike protocol identifiers, the skill
ID is part of the public package identity and should move with a future public
rename. If the public name changes again, update this file plus the README,
skill directory, `SKILL.md`, `agents/openai.yaml`, local discovery symlinks, and
tests, then run a case-insensitive search for the previous name. Treat every
remaining match as either an intentional compatibility identifier or a missed
public reference.
