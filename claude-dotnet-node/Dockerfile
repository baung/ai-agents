FROM claude-box
USER root

# .NET 8 SDK via the official install script; Microsoft's Debian apt repo has no arm64 packages
RUN apt-get update && apt-get install -y --no-install-recommends libicu-dev \
 && rm -rf /var/lib/apt/lists/* \
 && curl -fsSL https://dot.net/v1/dotnet-install.sh | bash -s -- --channel 8.0 --install-dir /usr/share/dotnet \
 && ln -s /usr/share/dotnet/dotnet /usr/local/bin/dotnet

# Latest Node.js 22 from the official image; agent-update pulls it so the tag stays current
COPY --from=node:22-bookworm-slim /usr/local/bin/node /usr/local/bin/
COPY --from=node:22-bookworm-slim /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s ../lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
 && ln -s ../lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

USER vscode
ENV DOTNET_ROOT=/usr/share/dotnet \
    DOTNET_CLI_TELEMETRY_OPTOUT=1 \
    PATH="/home/vscode/.dotnet/tools:${PATH}"
RUN dotnet tool install --global dotnet-ef --version "8.*"
