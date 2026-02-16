# cmd and entrypoint

## CMD (Command) Concept

**What is CMD?**\
CMD stands for "command" and generally refers to an instruction or directive given to a computer program or system to perform a specific task.

Real-time scenario:\
Imagine you're using a smartphone or computer. When you want to open an application, you tap or click on its icon — this action is like giving a command to the device to "start this program." Similarly, in programming or operating systems, commands are instructions that tell the computer what to do.

In a programming context, a CMD could be a command line instruction like:

* `copy file1.txt file2.txt` — which copies a file from one name to another.
* `start app.exe` — which tells the system to launch an application.

***

## Entry Point Concept

**What is an Entry Point?**\
The entry point is where a program begins execution. It’s like the front door of a building — when you enter, you start your journey inside. Similarly, a program has one designated starting place from which it begins running its code.

Real-time scenario:\
Think of watching a movie. The entry point is like the start button on your streaming service. Without pressing "play," the movie doesn’t begin. When you hit "play," the movie starts from the very first scene — that’s the entry point.

***

{% stepper %}
{% step %}
### Demo plan for students — quick steps

1. Explain the difference (CMD vs ENTRYPOINT) with analogies.
2. Show simple Dockerfile examples (CMD-only, ENTRYPOINT-only, ENTRYPOINT+CMD).
3. Demonstrate runtime behavior with `docker run` overrides.
4. Introduce build arguments (ARG) and show how they are used at build time and optionally converted to ENV.
5. Show a combined example (ARG + ENTRYPOINT + CMD) and run variations.
6. Cover common pitfalls and troubleshooting.

Use the following Dockerfiles and runtime commands during the live demo.
{% endstep %}

{% step %}
### Concepts recap (short)

* ENTRYPOINT sets the executable that will always run. Arguments passed in `docker run` are appended to ENTRYPOINT.
* CMD sets default arguments or a default command and can be overridden by passing commands to `docker run`.
* If ENTRYPOINT is not set, CMD acts as the command executed by the container.
* ARG (build arguments) exist only at build time unless converted to ENV.
{% endstep %}

{% step %}
### Examples to run (copy these into files and build/run)

Example 1: Using CMD only

Dockerfile (cmd-only):

```
FROM ubuntu
CMD ["echo", "Hello from CMD"]
```

Build and run:

```
docker build -t demo-cmd .
docker run demo-cmd
# Override CMD:
docker run demo-cmd echo "Overriding CMD"
```

Expected behavior: default runs `echo "Hello from CMD"`. Passing a command to `docker run` replaces CMD.
{% endstep %}

{% step %}
### More examples

Example 2: Using ENTRYPOINT only

Dockerfile (entrypoint-only):

```
FROM alpine
ENTRYPOINT ["ls"]
```

Build and run:

```
docker build -t demo-entrypoint .
docker run demo-entrypoint
# Pass args appended to ENTRYPOINT:
docker run demo-entrypoint -alh
```

Expected behavior: container runs `ls` by default; runtime args are appended to `ls`.

Example 3: ENTRYPOINT + CMD

Dockerfile (entrypoint+cmd):

```
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello from ENTRYPOINT and CMD"]
```

Build and run:

```
docker build -t demo-both .
docker run demo-both
# Override CMD argument but keep ENTRYPOINT:
docker run demo-both "Override argument"
```

Expected behavior: ENTRYPOINT is fixed (`echo`); CMD provides default argument that can be overridden by `docker run` parameters.
{% endstep %}

{% step %}
### Build arguments (ARG) — short demo

Dockerfile using ARG:

```
ARG APP_VERSION=1.0
FROM ubuntu

RUN echo "Building version $APP_VERSION"

ENTRYPOINT ["echo"]
CMD ["Default message"]
```

Build with custom ARG:

```
docker build --build-arg APP_VERSION=2.0 -t myapp:latest .
```

Notes:

* ARG values are available only during build.
*   To make a build-time value available at runtime, convert ARG into ENV:

    ```
    ARG VERSION=1.0
    ENV VERSION=$VERSION
    ```
{% endstep %}

{% step %}
### Combined practical example (ARG + ENTRYPOINT + CMD)

Dockerfile (Python web app example from the content):

```
ARG PYTHON_VERSION=3.11
ARG ENVIRONMENT=production
ARG APP_PORT=8000

FROM python:${PYTHON_VERSION}-slim

ENV ENVIRONMENT=$ENVIRONMENT
ENV APP_PORT=$APP_PORT

RUN if [ "$ENVIRONMENT" = "development" ]; then \
        pip install debugpy pytest; \
    fi

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

ENTRYPOINT ["python"]
CMD ["app.py", "--port", "$APP_PORT"]
```

Build and run examples:

```
# Build for production (default)
docker build -t myapp .

# Build for development with debugging tools
docker build -t myapp-dev --build-arg ENVIRONMENT=development .

# Run default: python app.py --port 8000
docker run -p 8000:8000 myapp

# Run different script while keeping ENTRYPOINT (python):
docker run myapp manage.py migrate

# Run a python one-liner:
docker run -it myapp -c "import sys; print(sys.version)"
```

Behavior: ENTRYPOINT ensures `python` is the executable; CMD provides default script and args. Runtime arguments replace CMD but are appended to ENTRYPOINT.
{% endstep %}
{% endstepper %}

***

## Correlation between CMD, ENTRYPOINT and Build Args

* ENTRYPOINT sets the executable.
* CMD provides default arguments to ENTRYPOINT.
* Arguments provided in `docker run` override CMD but append to ENTRYPOINT.
* ARG is for build-time configuration; not available at runtime unless converted to ENV.

***

## Troubleshooting (expandable)

<details>

<summary>Build args not available at runtime</summary>

Issue: ARG values disappear in running containers.\
Solution: Convert ARG to ENV if you need values at runtime.

Wrong:

```
ARG VERSION=1.0
CMD ["sh", "-c", "echo Version: $VERSION"]
```

Correct:

```
ARG VERSION=1.0
ENV VERSION=$VERSION
CMD ["sh", "-c", "echo Version: $VERSION"]
```

</details>

<details>

<summary>Cannot override container behavior</summary>

Issue: Users can't customize container execution.\
Solution: Use ENTRYPOINT + CMD pattern instead of CMD alone.

Less flexible:

```
CMD ["./app", "--config=prod", "--verbose"]
```

More flexible:

```
ENTRYPOINT ["./app"]
CMD ["--config=prod", "--verbose"]
```

This lets users override defaults while keeping the main executable fixed.

</details>

***

## Summary for students

* CMD: Default command/arguments to run. Can be overridden at runtime. (Analogy: default coffee machine button.)
* ENTRYPOINT: Defines the fixed executable to run. Runtime args append to it. (Analogy: a car engine that always starts.)
* ARG: Build-time variables for customizing the image build. Not persisted to runtime unless converted to ENV. (Analogy: choosing ingredients before cooking.)

Use the ENTRYPOINT + CMD pattern for predictable, flexible images; use ARG to parameterize builds and convert to ENV when runtime access is needed.

If you want, I can produce short slides or a step-by-step live demo script with exact terminal commands for your lesson.
