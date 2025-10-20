---
layout: post
title: "Running Claude inside Docker (Part 1)"
date: 2025-10-20
categories: ai
---
I decided I'd like to try Claude Code, but was a little wary of immediately giving it access to my server content.
Therefore I came to the conclusion that it'd be nice to run it inside Docker and keep it's capability restricted.

This was accomplished easily enough by creating a `Dockerfile` with the below content:
```
FROM node:24
WORKDIR /app
RUN npm install -g @anthropic-ai/claude-code
RUN useradd -m claude_runner
RUN chown -R claude_runner:claude_runner /app
USER claude_runner
ENTRYPOINT ["claude"]
```

I was then able to build a Docker image from the current directory as follows:
```
docker build -t claude-djhills .
```

And run the created container as follows:
```
docker run -ti claude-djhills
```

At this point I was welcomed to Claude with a warning that it has access to all of my files in the `/app` directory, but of course this directory actually only exists inside the Docker container.  

![Image](/assets/images/claude_ready.png)  

Mission accomplished!

