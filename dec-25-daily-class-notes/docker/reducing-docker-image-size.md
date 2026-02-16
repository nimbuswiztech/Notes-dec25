# reducing docker image size

{% stepper %}
{% step %}
### Choose a Minimal Base Image

Using a lean starting point removes unnecessary components from day one.

* **Alpine Linux**\
  – Alpine images are typically 5–7 MB versus hundreds of MB in Debian/Ubuntu.\
  – Caveat: some binaries rely on GNU libc; you may need Alpine’s `glibc` package or choose an Alpine-compatible build.
* **Distroless or Scratch**\
  – `scratch` starts from an empty image; ideal for statically compiled Go, Rust, or C++ binaries.\
  – Google’s Distroless images include only your language runtime and application code, omitting shell and package managers.
{% endstep %}

{% step %}
### Leverage Multi-Stage Builds

Break your Dockerfile into **build** and **runtime** stages to discard build-time dependencies.

{% code title="Dockerfile (multi-stage build)" %}
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o myapp

# Runtime stage
FROM scratch
COPY --from=builder /app/myapp /myapp
ENTRYPOINT ["/myapp"]
```
{% endcode %}

Only the final compiled binary is copied; all Go toolchain and source files are left behind.
{% endstep %}

{% step %}
### Minimize Layers and Combine Commands

Every `RUN`, `COPY`, or `ADD` instruction creates a new layer. Fewer layers → smaller image.

* Combine related operations:

{% code title="Example: combine apt operations" %}
```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
    && rm -rf /var/lib/apt/lists/*
```
{% endcode %}

* Clean caches and lists in the same `RUN` step to avoid leaving them in intermediate layers.
{% endstep %}

{% step %}
### Clean Up Package Manager Caches

Remove unnecessary files immediately after installation:

* **APT (Debian/Ubuntu):**\
  `rm -rf /var/lib/apt/lists/*`
* **YUM (CentOS/RHEL):**\
  `yum clean all && rm -rf /var/cache/yum`
* **apk (Alpine):**\
  `apk --no-cache add <packages>`
{% endstep %}

{% step %}
### Exclude Unneeded Files via .dockerignore

Use a `.dockerignore` file to prevent copying source control metadata, docs, tests, and local configs into the build context.

Example `.dockerignore`:

{% code title=".dockerignore" %}
```
.git
node_modules
*.md
*.log
tests/
```
{% endcode %}
{% endstep %}

{% step %}
### Use Language-Specific Best Practices

Tailor cleanup to your language ecosystem:

* **Node.js**
  * Install only production dependencies:\
    `npm ci --production` or `yarn install --production`
  * Prune devDependencies:\
    `npm prune --production`
* **Python**
  * Use a requirements file with only needed packages.
  * Use `pip install --no-cache-dir -r requirements.txt` to skip pip cache.
* **Java**
  * Build a “fat” JAR outside the image, then use `FROM openjdk:jre` instead of `jdk`.
  * Consider tools like JLink or GraalVM Native Image to produce a minimal runtime.
{% endstep %}

{% step %}
### Remove Build-Only Tools and Files

In multi-stage builds, ensure compilers, build caches, and documentation are left in earlier stages:

* Delete source archives, `.o` files, and intermediate artifacts.
* Avoid copying the entire source directory into the final image.
{% endstep %}

{% step %}
### Analyze and Optimize Image Contents

Use Docker’s built-in tools to inspect and prune:

* `docker image ls` and `docker history <image>`
* `docker image inspect <image>`
* [Dive](https://github.com/wagoodman/dive): interactive exploration of each layer’s contents.

Identify large files or directories that can be excluded or compressed.
{% endstep %}

{% step %}
### Compress or Archive Assets

Where appropriate, bundle static assets (e.g., web assets) into compressed archives and extract at runtime, or serve them via an external CDN to avoid embedding heavy assets in the image.
{% endstep %}

{% step %}
### Automate Image Slimming

Integrate regular image audits and size checks into your CI pipeline:

* Fail builds if image exceeds a size threshold.
* Report layer sizes and trending growth over time.
{% endstep %}
{% endstepper %}
