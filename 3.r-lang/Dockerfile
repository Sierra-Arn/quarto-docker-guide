# =====================================================
# Stage 1: System Dependencies
# =====================================================

# Use Ubuntu 24.04 as foundation for explicit dependency control
# - Starts clean to avoid hidden pre-installed packages
# - Forces transparent declaration of all components
FROM ubuntu:24.04

# NOTE: R Installation Strategy
# R can be installed via apt-get (recommended for latest versions and security updates),
# but we use conda-forge here for development convenience:
# - Pre-compiled packages save significant build time
# - Simplified dependency management across R packages
# - Consistent environment isolation
# For production deployments, consider apt-get installation for official Ubuntu packages

# Set UTF-8 locale to prevent common container issues
# - Eliminates "unsupported locale" errors in logs
# - Ensures proper stdin/stdout encoding
# - Guarantees correct interpretation of special characters
# - Uses C locale with UTF-8 rather than region-specific settings
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# Install base system utilities needed for Micromamba
# - ca-certificates: SSL certificate verification
# - curl: Download tool for micromamba
# - tar & bzip2: Archive handling for micromamba binary
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    tar \
    bzip2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install micromamba (conda alternative) system-wide
# - Binary is placed in /usr/local/bin for global access
# - Version pinned for reproducibility
ARG MICROMAMBA_VERSION=1.5.8
RUN curl -Ls "https://micro.mamba.pm/api/micromamba/linux-64/${MICROMAMBA_VERSION}" \
    | tar -xjv bin/micromamba \
    && mv bin/micromamba /usr/local/bin/ \
    && chmod +x /usr/local/bin/micromamba

# =====================================================
# Stage 2: User Configuration
# =====================================================

# Configure runtime parameters
# - Non-root user for security best practices
# - Dedicated conda prefix for environment isolation
ARG DEV_USER=user
ENV MAMBA_ROOT_PREFIX=/opt/conda \
    ENV_NAME=dev

# Create application user with ownership of key directories
# - Home directory required for shell initialization files
# - Conda prefix directory for package installations
# - Explicit permissions prevent privilege escalation risks
RUN useradd --create-home --shell /bin/bash ${DEV_USER} \
    && mkdir -p ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} /home/${DEV_USER}

# Switch to unprivileged user context
USER ${DEV_USER}
WORKDIR /home/${DEV_USER}

# =====================================================
# Stage 3: R Environment Setup
# =====================================================

# Initialize micromamba shell integration
# - Shell hooks enable 'micromamba activate' in bash sessions
# - Automatic environment activation via .bashrc
# - Ensures consistent environment loading across sessions
RUN micromamba shell init --shell bash --root-prefix="${MAMBA_ROOT_PREFIX}" && \
    echo 'eval "$(micromamba shell hook --shell bash)"' >> ~/.bashrc && \
    echo "micromamba activate ${ENV_NAME}" >> ~/.bashrc

# Create base R environment with essential packages
# - conda-forge channel for community-maintained packages
# - r-base: Core R installation
# - knitr: Dynamic report generation engine for R
# - knitr installation validates package management and dependency resolution
RUN micromamba create -y -n ${ENV_NAME} -c conda-forge \
    r-base \
    r-knitr \
    && micromamba clean --all -y

# Global environment configuration for R integration
# - PATH: Update container PATH to include environment binaries
# - R_HOME: Points to R installation directory
# - R_LIBS_USER: Defines user library path for package installations
ENV PATH="${MAMBA_ROOT_PREFIX}/envs/${ENV_NAME}/bin:${PATH}" \
    R_HOME="${MAMBA_ROOT_PREFIX}/envs/${ENV_NAME}/lib/R" \
    R_LIBS_USER="${MAMBA_ROOT_PREFIX}/envs/${ENV_NAME}/lib/R/library"

# Configure R user preferences and library paths
# - CRAN mirror set to cloud.r-project.org for reliability
# - Library paths configured to use conda environment
# - .Rprofile ensures settings persist across R sessions
RUN echo 'options(repos = c(CRAN = "https://cloud.r-project.org"))' >> ~/.Rprofile && \
    echo '.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))' >> ~/.Rprofile

# =====================================================
# Stage 4: Verification and Runtime Configuration
# =====================================================

# Final CMD combines version checks with persistent shell
# - First bash: Initializes new shell process
# - Flag -i: Forces interactive mode to load ~/.bashrc
# - Flag -c: Executes string-based commands then continues
# - && chain: Runs commands sequentially (version checks)
# - Final bash: Maintains active shell after validations
CMD ["bash", "-i", "-c", "micromamba --version && R --version && bash"]