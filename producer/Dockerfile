  FROM python:3.8-slim

  ENV PYTHONDONTWRITEBYTECODE=1 \
      PYTHONUNBUFFERED=1

  # Create a non-root user
  RUN adduser --disabled-password --gecos '' appuser

  WORKDIR /app
  COPY producer.py /app/producer.py

  # Install dependencies
  RUN pip install --no-cache-dir kafka-python

  # Change to non-root user
  USER appuser

  CMD ["python", "producer.py"]
