I have about 40 route files under `src/routes/`. There is no shared auth middleware — this
codebase grew by acquisition and each route hand-rolls its own check, so they are genuinely
heterogeneous and no single grep characterises them. Every one needs reading on its own terms
to decide whether it is properly guarded. It is a read-only audit; nothing gets written.

How should I run this?
