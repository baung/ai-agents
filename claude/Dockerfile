FROM agent-base
# latest, stable, or a version like 2.1.0; changing it busts the cache for this layer
ARG VERSION=latest
RUN curl -fsSL https://claude.ai/install.sh | bash -s -- "$VERSION" \
 && mkdir -p /home/vscode/.claude
ENV PATH="/home/vscode/.local/bin:${PATH}" \
    CLAUDE_CONFIG_DIR="/home/vscode/.claude"
CMD ["claude"]
