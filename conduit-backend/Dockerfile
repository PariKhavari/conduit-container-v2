# ---------- Stage 1: Builder ----------
FROM python:3.6-slim-buster AS builder

WORKDIR /app

# Debian Buster ist EOL -> Standard-Repos existieren nicht mehr, auf Archiv umleiten
RUN sed -i \
    -e 's|deb.debian.org/debian|archive.debian.org/debian|g' \
    -e 's|security.debian.org/debian-security|archive.debian.org/debian-security|g' \
    -e '/buster-updates/d' \
    /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt


# ---------- Stage 2: Runtime ----------
FROM python:3.6-slim-buster AS runtime

RUN sed -i \
    -e 's|deb.debian.org/debian|archive.debian.org/debian|g' \
    -e 's|security.debian.org/debian-security|archive.debian.org/debian-security|g' \
    -e '/buster-updates/d' \
    /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends libpq5 \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

COPY --from=builder /install /usr/local

# Kompletten Code kopieren, DANACH erst entrypoint.sh reparieren
# (wichtig: sed-Fix muss NACH diesem COPY passieren, sonst wird er überschrieben!)
COPY --chown=appuser:appuser . .
RUN sed -i 's/\r$//' entrypoint.sh \
    && chmod +x entrypoint.sh

ENV DJANGO_SETTINGS_MODULE=conduit.settings \
    PYTHONUNBUFFERED=1 \
    PORT=8000

RUN mkdir -p /app/staticfiles && chown -R appuser:appuser /app/staticfiles

USER appuser

EXPOSE 8000

ENTRYPOINT ["/bin/sh", "/app/entrypoint.sh"]
CMD ["gunicorn", "conduit.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3"]