# syntax=docker/dockerfile:1

# Comments are provided throughout this file to help you get started.
# If you need more help, visit the Dockerfile reference guide at
# https://docs.docker.com/go/dockerfile-reference/

# Want to help us make this template better? Share your feedback here: https://forms.gle/ybq9Krt8jtBL3iCk7

ARG PYTHON_VERSION=3.12.9
FROM python:${PYTHON_VERSION}-slim as base

# Prevents Python from writing pyc files.
ENV PYTHONDONTWRITEBYTECODE=1

# Keeps Python from buffering stdout and stderr to avoid situations where
# the application crashes without emitting any logs due to buffering.
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Create a non-privileged user that the app will run under.
# See https://docs.docker.com/go/dockerfile-user-best-practices/
# ARG UID=10001
# RUN adduser \
#     --disabled-password \
#     --gecos "" \
#     --home "/nonexistent" \
#     --shell "/sbin/nologin" \
#     --no-create-home \
#     --uid "${UID}" \
#     appuser

# Switch to root to install system dependencies
USER root
RUN apt-get update && apt-get install -y \
    wget \
    unzip \
    curl \
    firefox-esr \
    libx11-xcb1 \
    libxtst6 \
    libxrender1 \
    libdbus-glib-1-2 \
    libgtk-3-0 \
    libasound2 \
    fonts-liberation \
    libgl1-mesa-dri \
    libpci3 \
    cron \
    vim \
    && rm -rf /var/lib/apt/lists/*

# Configure Fontconfig to avoid cache errors
RUN mkdir -p /tmp/cache/fontconfig && chmod 777 /tmp/cache/fontconfig
ENV FONTCONFIG_PATH=/tmp/cache/fontconfig

# Download and install GeckoDriver
RUN case $(dpkg --print-architecture) in \
    amd64) ARCH=linux64 ;; \
    arm64) ARCH=linux-aarch64 ;; \
    *) echo "Unsupported architecture" && exit 1 ;; \
    esac && \
    wget -O /tmp/geckodriver.tar.gz https://github.com/mozilla/geckodriver/releases/download/v0.35.0/geckodriver-v0.35.0-$ARCH.tar.gz && \
    tar -xvzf /tmp/geckodriver.tar.gz -C /usr/local/bin/ && \
    chmod +x /usr/local/bin/geckodriver && \
    rm /tmp/geckodriver.tar.gz

# Install Python dependencies
COPY requirements.txt /app/
RUN python -m pip install --no-cache-dir -r requirements.txt

# Ensure permissions for SQLite database and source code
RUN mkdir -p /app/webscraper_project && \
    touch /app/webscraper_project/db.sqlite3 && \
    chmod -R 777 /app/webscraper_project && \
    chmod -R 777 /app

    # Crear directorios necesarios y asignar permisos para cron
RUN mkdir -p /var/run /var/log && \
    chmod 0755 /var/run /var/log && \
    touch /var/run/crond.pid && \
    chmod 0644 /var/run/crond.pid && \
    touch /var/log/cron.log && \
    chmod 0644 /var/log/cron.log

# Copia el archivo local cronfile (que contiene las reglas de cron) al directorio /etc/cron.d/ con el nombre scrape-cron.
COPY cronfile /etc/cron.d/scrape-cron

# Dar permisos adecuados al cronfile
RUN chmod 0644 /etc/cron.d/scrape-cron

# Registrar el cronjob
RUN crontab /etc/cron.d/scrape-cron

# Create screenshots directory and set permissions
RUN mkdir -p /app/screenshots && chmod 777 /app/screenshots

# Create volume for screenshots para acceder a ellas
VOLUME ["/app/screenshots"]

# Copy the source code into the container.
COPY . .

# Expose the port that the application listens on.
EXPOSE 8000

# Run the application.
#CMD python3 django_webscraper_project/manage.py scrape

# Default command to keep the container running
#CMD ["tail", "-f", "/dev/null"]

# Start the cron service in the foreground
CMD ["cron", "-f"]