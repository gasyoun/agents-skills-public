## Serial multi-profile assessment (documented, NOT exercised here)

To assess several profiles, run Steps 1+2 **serially, one profile at a time**, each with its **own distinct `config_dir_fingerprint`** and its own health+canary pair; never interleave or share a fingerprint. Each slot gets its own ration clock (Step 1) and its own receipt (Step 2).

