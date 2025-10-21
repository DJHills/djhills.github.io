---
layout: post
title: "Running Claude inside Docker (Part 2)"
date: 2025-10-21
categories: ai
---
Now that I had Claude Code up and running, I wondered if I could have used Claude itself to generate the Dockerfile, and if so, what improvements it might have over the version I wrote by hand.

After a couple of prompts to Claude explaining my requirements to not only create a Dockerfile, but to also have Python available within the environment, it had managed to generate the below `Dockerfile` (including the comments shown), which is significantly better than the version I crafted by hand:

```
FROM python:3.14

# Install Node.js (required for Claude Code)
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Claude Code globally via npm
RUN npm install -g @anthropic-ai/claude-code

# Set up Python environment
# Install common Python development tools
RUN pip install --no-cache-dir --upgrade pip setuptools wheel

# Optional: Install common Python packages
# Uncomment and modify based on your needs
# RUN pip install --no-cache-dir \
#     numpy \
#     pandas \
#     requests \
#     pytest

# Set working directory
WORKDIR /workspace

# Create a non-root user for better security
RUN useradd -m -s /bin/bash developer && \
    chown -R developer:developer /workspace

USER developer

# Default command
CMD ["/bin/bash"]
```

As before I was then able to build a Docker image from as follows:
```
docker build -t claude-djhills .
```

And run the created container as follows:
```
docker run -ti claude-djhills
```

This is the version I'm currently using for further experimentation.

