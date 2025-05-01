# DIP - Docker Integration Platform

DIP is a lightweight command-line tool designed to simplify Docker development workflows. It provides a unified interface for common Docker and docker-compose operations, making it easier to manage Docker-based development environments.

## Features

- Simple, intuitive commands for common Docker operations
- Automatic project detection and configuration
- Support for custom commands tailored to your project
- Database import/export utilities
- Health status monitoring for services
- Resource usage statistics
- Integration with common development tools

## Installation

### Prerequisites

- Bash shell
- Docker and docker-compose
- Git (for updates)

### Quick Install

```bash
# Clone the repository
git clone https://github.com/sy2nyk/dip.git

# Make the script executable
chmod +x dip/dip

# Install DIP (this will copy the script to ~/.local/bin/)
./dip/dip update
