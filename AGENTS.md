# BTrace Repository Rules

- Follow the checked-out source and existing module conventions; never overwrite a newer file with a vendored SDK copy.
- Preserve upstream version, Maven coordinates, release configuration and unrelated work. Publishing or pushing requires an explicit user request.
- Keep Android compatibility patches confined to rhea-inhouse, preserving original symbol resolution before fallbacks. Do not copy host-side tracing or SVGA player logic into the SDK.
- Document upstream revision, local changes, device evidence and unverified build/ABI boundaries in ANDROID_COMPATIBILITY.md.
- Build outputs belong to Gradle build directories and may be cleaned by their creator. Preserve source and original device evidence.
- Keep AGENTS.md in English. Local code comments may be Chinese.
