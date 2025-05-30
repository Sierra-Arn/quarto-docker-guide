# =====================================================
# Stage 1: System Dependencies
# =====================================================

# Use Ubuntu 24.04 as foundation for explicit dependency control
# - Starts clean to avoid hidden pre-installed packages
# - Forces transparent declaration of all components
FROM ubuntu:24.04

# Set UTF-8 locale to prevent common container issues
# - Eliminates "unsupported locale" errors in logs
# - Ensures proper stdin/stdout encoding
# - Guarantees correct interpretation of special characters
# - Uses C locale with UTF-8 rather than region-specific settings
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# Install base system utilities needed for Juliaup
# - ca-certificates: SSL certificate verification
# - curl: Download tool for juliaup installer
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# =====================================================
# Stage 2: User Configuration
# =====================================================

# Configure runtime parameters
# - Non-root user for security best practices
ARG DEV_USER=user

# Create application user with ownership of key directories
# - Home directory required for shell initialization files
# - Explicit permissions prevent privilege escalation risks
RUN useradd --create-home --shell /bin/bash ${DEV_USER} && \
    chown -R ${DEV_USER}:${DEV_USER} /home/${DEV_USER}

# Switch to unprivileged user context
USER ${DEV_USER}
WORKDIR /home/${DEV_USER}

# =====================================================
# Stage 3: Julia Environment Setup
# =====================================================

# Install juliaup (Julia version manager) in user home directory
# - Installs to user space for isolation
# - Flag -y provides automatic consent for installation
# - Custom path ensures predictable location
RUN curl -fsSL https://install.julialang.org | sh -s -- -y --path ~/.juliaup

# Update container PATH to include juliaup binaries 
ENV PATH="/home/${DEV_USER}/.juliaup/bin:${PATH}"

# Install specific Julia version for reproducibility
# - Version pinned for predictable behavior
# - Set as default version for automatic activation
RUN juliaup add 1.11.5 && \
    juliaup default 1.11.5

# Configure Julia environment variables
# - JULIA_PROJECT="@." automatically activates project in current directory
# - Added to .bashrc for persistence across sessions
RUN echo 'export JULIA_PROJECT="@."' >> ~/.bashrc

# Install base Julia packages and precompile
# - Example: demonstration package for functionality verification
# - Precompilation reduces startup time for subsequent usage
RUN julia -e 'using Pkg; Pkg.add(["Example"])' && \
    julia -e 'using Pkg; Pkg.precompile()'

# =====================================================
# Stage 4: Verification and Runtime Configuration
# =====================================================

# Final CMD combines version checks with persistent shell
# - First bash: Initializes new shell process
# - Flag -i: Forces interactive mode to load ~/.bashrc
# - Flag -c: Executes string-based commands then continues
# - Final bash: Maintains active shell after validations
CMD ["bash", "-i", "-c", "julia --version && bash"]