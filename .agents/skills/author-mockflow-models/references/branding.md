<!-- Centralizes public naming and stable compatibility identifiers for the skill package. -->
# Branding and compatibility

The public product name is `graphRun`, with a lowercase `graph` and uppercase
`R`. Use it in the README, skill description and title, and agent display
metadata. Keep deeper operating guidance product-neutral by referring to “the
MCP”, “the server”, or “the product”; this limits future renaming to the public
entry points and this file.

Do not rename compatibility-sensitive identifiers during a public rebrand:

- skill ID and directory: `author-mockflow-models`;
- schema and contract names such as `mockflow.mcp.*`;
- MCP resource URIs beginning with `mockflow://`;
- opaque references and local placeholders such as `mfref2.*` and
  `mcp-local-*`;
- package, environment, header, extension, and media-type identifiers that
  intentionally retain `mockflow`.

The repository locator is `graphrun/skills`. If the public name changes again,
update this file plus the README, `SKILL.md`, and `agents/openai.yaml`, then run
a case-insensitive search for the previous name. Treat every remaining match as
either an intentional compatibility identifier or a missed public reference.
