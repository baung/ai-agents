FROM agent-base
RUN curl -fsSL https://claude.ai/install.sh | bash && mkdir -p /home/vscode/.claude
ENV PATH="/home/vscode/.local/bin:${PATH}" \
    CLAUDE_CONFIG_DIR="/home/vscode/.claude"
CMD ["claude"]
