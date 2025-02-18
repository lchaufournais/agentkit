# Utilisez l'image officielle allégée de FastAPI (basée sur Uvicorn/Gunicorn)
FROM tiangolo/uvicorn-gunicorn-fastapi:python3.10-slim-2022-11-25

# Variables d'environnement
ENV PYTHONUNBUFFERED=1 \
    POETRY_HOME="/opt/poetry" \
    PATH="/opt/poetry/bin:$PATH" \
    PYTHONPATH=/code

WORKDIR /code

# Installer les dépendances système et Poetry en une seule couche
RUN apt-get update && \
    apt-get install -y curl && \
    curl -sSL https://install.python-poetry.org | POETRY_HOME=/opt/poetry python && \
    ln -s /opt/poetry/bin/poetry /usr/local/bin/poetry && \
    poetry config virtualenvs.create false && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Copier en premier les fichiers de dépendances pour bénéficier du cache
COPY app/pyproject.toml app/poetry.lock* /code/

# Installation des dépendances (argument pour inclure ou non les dev)
ARG INSTALL_DEV=false
RUN if [ "$INSTALL_DEV" = "true" ]; then \
      poetry install --with dev --no-root; \
    else \
      poetry install --no-root --without dev; \
    fi

# Copier le reste du code de l’application
COPY app /code/app

EXPOSE 9090

# Commande par défaut (vous pouvez aussi utiliser docker-compose pour définir la commande)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "9090", "--reload", "--workers", "1"]
