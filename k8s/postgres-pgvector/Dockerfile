# ── Stage 1: grab the compiled extension ────────────────────────────────
FROM pgvector/pgvector:pg17 AS builder

# ── Stage 2: Bitnami PostgreSQL + pgvector ──────────────────────────────
FROM bitnami/postgresql:17

# Copy .so and control/sql files to the locations Bitnami expects
COPY --from=builder /usr/lib/postgresql/17/lib/vector.so \
                    /opt/bitnami/postgresql/lib/
COPY --from=builder /usr/share/postgresql/17/extension/*vector* \
                    /opt/bitnami/postgresql/share/extension/
