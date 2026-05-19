# Stage 1 — builder
FROM python:3.11-slim AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

# Stage 2 — final image
FROM python:3.11-slim

# Non-root user — security best practice
RUN addgroup --gid 1001 appgroup \
    && adduser --uid 1001 --gid 1001 \
       --disabled-password --no-create-home appuser

WORKDIR /app

# Copy installed packages from builder stage
COPY --from=builder /usr/local/lib/python3.11/site-packages \
     /usr/local/lib/python3.11/site-packages
COPY --from=builder /usr/local/bin/gunicorn \
     /usr/local/bin/gunicorn

# Copy app code
COPY app.py .

# Set ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD python -c \
    "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')"

CMD ["gunicorn", "--bind", "0.0.0.0:8080", "--workers", "2", "app:app"]