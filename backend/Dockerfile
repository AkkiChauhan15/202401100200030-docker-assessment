FROM python:3.13.18-slim-bookworm

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

RUN groupadd --system --gid 10001 festpass \
    && useradd --system --uid 10001 --gid festpass \
        --no-create-home --shell /usr/sbin/nologin festpass

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --requirement requirements.txt

ARG APP_VERSION=dev
ENV APP_VERSION=${APP_VERSION}

COPY --chown=festpass:festpass app.py .

USER 10001:10001

EXPOSE 5000

CMD ["python", "app.py"]
