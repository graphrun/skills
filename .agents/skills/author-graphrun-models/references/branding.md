<!-- Centralizes public naming and stable compatibility identifiers for the skill package. -->
# Branding and compatibility

The public product name is `Graphrun`, with an uppercase `G` and the remaining
letters lowercase. Use this spelling whenever naming the product, including
the README, skill instructions, reference guides, and agent display metadata.

Do not rename compatibility-sensitive identifiers during a public rebrand:

- schema and contract identifiers;
- MCP resource URIs;
- opaque references and local placeholders;
- package, environment, header, extension, and media-type identifiers.

Use exact identifiers from live tool schemas and returned resource links,
even when their spelling differs from the public product name. Do not invent
Graphrun-prefixed replacements or construct resource URIs from the brand name.

The repository locator is `graphrun/skills`, and the current skill ID and
directory are `author-graphrun-models`. Keep these lowercase installation and
invocation identifiers unchanged when correcting product capitalization.
