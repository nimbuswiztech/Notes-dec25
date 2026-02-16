# docker best practices

### General Docker Best Practices

* **Use Multi-Stage Builds:** Helps reduce image size and keep builds clean by separating build dependencies from the final runtime image.
* **Choose Official and Minimal Base Images:** Start with official, verified images based on your stack. Where possible, pick minimal base images to lower the surface area for security issues and decrease overall image size.
* **Order Dockerfile Commands for Caching:** Place commands that change infrequently (like dependency installation) near the top, and commands that change frequently (like `COPY` or `ADD` for your application code) below them. This leverages Docker's build cache, making rebuilds faster.
* **Minimize the Number of Layers:** Combine commands where possible using `&&` to limit the image's layer count, which keeps images leaner.
* **Prefer COPY over ADD:** Use `COPY` for transferring files as it is more predictable and explicit. Reserve `ADD` for its advanced features when necessary, such as unpacking local `.tar` files.
* **Use .dockerignore:** Maintain a `.dockerignore` file to prevent unnecessary files and directories from being included in the build context, improving build speed and creating smaller images.
* **Run as a Non-Root User:** Avoid running processes in containers as `root`; use the `USER` directive to create and switch to a dedicated unprivileged user.
* **One Process per Container:** Stick to the microservices principle—have each container execute a single main process where possible. Use orchestration tools if you need to run multiple services.
* **Include HEALTHCHECK Instructions:** Define a `HEALTHCHECK` in your Dockerfile so Docker can determine if your container is running as expected.
* **Don’t Store Secrets in Images:** Never embed passwords, API keys, or secrets directly within images. Use secure secrets management tools.

### Security-Focused Best Practices

* **Keep Docker and Host Up-to-Date:** Regularly update both Docker and the underlying OS to patch vulnerabilities.
* **Do Not Expose Docker Daemon Socket:** Never mount the Docker socket inside containers or expose it unnecessarily; this can give containers root access to the host.
* **Prevent Privilege Escalation:** Run containers with `--security-opt=no-new-privileges` to block processes from gaining additional privileges.
* **Limit Resources:** Set resource constraints (CPU, memory, number of processes) on containers to prevent abuse and DoS attacks.
* **Set Filesystem and Volumes to Read-Only:** Use the `--read-only` flag and mount data volumes as read-only unless write access is required.
* **Scan Images for Vulnerabilities:** Regularly scan images with tools like Docker scan or Trivy before deploying them.
* **Sign and Verify Images:** Use Docker Content Trust (DCT) or similar services to sign images and ensure authenticity.

### Build and Deployment Tips

* **Use Version Tags for Images:** Always specify image versions, not `latest`, for dependable deployments.
* **Log to stdout/stderr:** Docker captures logs from the main process; ensure your application logs go to stdout or stderr for easier aggregation and troubleshooting.
* **Limit Port Exposure:** Only expose or publish necessary ports to reduce attack surfaces.

By following these practices, you can ensure your Docker workflows are secure, efficient, and maintainable, suitable for both development and production environments.
