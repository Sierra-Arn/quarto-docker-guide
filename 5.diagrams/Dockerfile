# =====================================================
# Stage 1: System Dependencies
# =====================================================

# Quarto has native support for embedding Mermaid and Graphviz diagrams
# HTML export requires no additional dependencies - diagrams render natively in browsers
# PDF export requires Chrome/Chromium for HTML-to-PDF conversion - this creates significant complications
# The complexity arises from Chrome's headless rendering requirements in containerized environments

FROM ubuntu:24.04

ARG DEV_USER=user \
    QUARTO_VERSION=1.8.11
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# Headless Chromium dependencies for diagram rendering and PDF generation
# Required for Quarto's diagram engines (Mermaid, Observable, etc.) and HTML-to-PDF conversion
# - libgobject-2.0-0: GObject introspection library for GTK applications
# - libnss3: Network Security Services library for secure connections
# - libatk1.0-0, libatk-bridge2.0-0: Accessibility toolkit libraries
# - libcups2: Common UNIX Printing System library
# - libdrm2: Direct Rendering Manager library for graphics
# - libgtk-3-0: GTK+ 3 graphical user interface library
# - libnspr4: Netscape Portable Runtime library
# - libgbm1: Generic Buffer Management library for graphics
# - libasound2-dev: Advanced Linux Sound Architecture development files
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    pandoc \
    libfontconfig1 \
    libgobject-2.0-0 \
    libnss3 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libcups2 \
    libdrm2 \
    libgtk-3-0 \
    libnspr4 \
    libnss3 \
    libgbm1 \
    libasound2-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN curl -sSL "https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.deb" -o quarto.deb \
    && dpkg -i quarto.deb \
    && rm quarto.deb

# =====================================================
# Stage 2: User Configuration
# =====================================================
RUN useradd --create-home --shell /bin/bash ${DEV_USER} \
    && mkdir -p ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} /home/${DEV_USER}

USER ${DEV_USER}
WORKDIR /home/${DEV_USER}

# =====================================================
# Stage 3: Quarto Tools Installation
# =====================================================
RUN bash -c "quarto install tool tinytex --no-prompt"

# WARNING!
# =====================================================
# Critical Chromium Architecture Changes and Docker Compatibility Issues
# =====================================================
# Chrome/Chromium has undergone fundamental architectural changes that create
# significant compatibility challenges in containerized environments:
#
# 1. Chromium Architecture Evolution:
#    - Introduction of --headless=new flag replacing legacy --headless mode
#    - New chrome-headless-shell binary for optimized headless operations
#    - Updated security sandbox requirements and process isolation models
#
# 2. Quarto Chromium Installation Limitations:
#    - 'quarto install chromium' installs outdated Chromium versions
#    - Legacy versions lack support for modern headless architecture
#    - Bundled Chromium may be incompatible with container runtime environments
#
# 3. System Package Manager Conflicts:
#    - Installing chromium via apt-get creates Quarto rendering failures
#    - System-installed Chromium triggers snap dependency requirements
#    - Quarto expects specific Chromium installation paths and configurations
#    - Error messages demand snap-based Chromium installation for compatibility
#
# 4. Snap Installation Anti-Pattern in Docker:
#    - Snap packages require systemd and additional daemon processes
#    - Container security models conflict with snap's privilege requirements
#    - Introduces unnecessary complexity and potential security vulnerabilities
#    - Violates Docker best practices for minimal, single-purpose containers
#
# 5. Puppeteer Integration Challenges:
#    - Quarto installs headless Chromium via Puppeteer automation framework
#    - Puppeteer's bundled Chromium often incompatible with Docker container environments
#    - Missing system libraries and sandbox restrictions cause runtime failures
#    - Container-specific library dependencies not included in Puppeteer bundles
# Recommended Solutions:
# - Use local installation outside Docker containers
RUN bash -c "quarto install chromium --no-prompt" 

# =====================================================
# Stage 4: Verification and Runtime Configuration
# =====================================================
CMD ["bash", "-i", "-c", "quarto --version && quarto check && bash"]