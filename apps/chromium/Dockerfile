ARG BASE_IMAGE=ghcr.io/m1k1o/neko/base:latest
FROM $BASE_IMAGE

#
# install neko chromium
RUN set -eux; \
    echo "deb http://ftp.de.debian.org/debian bookworm main" >> /etc/apt/sources.list; \
    apt-get update; \
    apt-get install -y --no-install-recommends chromium chromium-common chromium-sandbox openbox socat; \
    #
    # install widevine module (only for amd64)
    CHROMIUM_DIR="/usr/lib/chromium"; \
    ARCH=$(dpkg --print-architecture); \
    if [ "${ARCH}" = "amd64" ]; then \
        # https://commondatastorage.googleapis.com/chromeos-localmirror/distfiles/chromeos-lacros-arm64-squash-zstd-120.0.6098.0
        apt-get install -y --no-install-recommends unzip; \
        WIDEVINE_ARCH="x64"; \
        WIDEVINE_VERSION=$(wget --quiet -O - https://dl.google.com/widevine-cdm/versions.txt | sort --version-sort | tail -n 1); \
        wget -O /tmp/widevine.zip "https://dl.google.com/widevine-cdm/${WIDEVINE_VERSION}-linux-${WIDEVINE_ARCH}.zip"; \
        mkdir -p "${CHROMIUM_DIR}/WidevineCdm/_platform_specific/linux_${WIDEVINE_ARCH}"; \
        unzip -p /tmp/widevine.zip LICENSE.txt > "${CHROMIUM_DIR}/WidevineCdm/LICENSE"; \
        unzip -p /tmp/widevine.zip manifest.json > "${CHROMIUM_DIR}/WidevineCdm/manifest.json"; \
        unzip -p /tmp/widevine.zip libwidevinecdm.so > "${CHROMIUM_DIR}/WidevineCdm/_platform_specific/linux_${WIDEVINE_ARCH}/libwidevinecdm.so"; \
        find "${CHROMIUM_DIR}/WidevineCdm" -type d -exec chmod 0755 '{}' \;; \
        find "${CHROMIUM_DIR}/WidevineCdm" -type f -exec chmod 0644 '{}' \;; \
        rm /tmp/widevine.zip; \
    else \
        echo "Widevine is not supported on ${ARCH}"; \
    fi; \
    #
    # install MCP Chrome extension
    apt-get install -y --no-install-recommends unzip wget nodejs npm; \
    #
    # install Chrome MCP
    mkdir -p /srv/mcp/chrome-mcp; \
    wget -O /tmp/chrome-mcp-server.zip "https://github.com/hangwin/mcp-chrome/releases/download/v0.0.6/chrome-mcp-server-0.0.6.zip"; \
    unzip /tmp/chrome-mcp-server.zip -d /srv/mcp/chrome-mcp/extension; \
    chmod -R 755 /srv/mcp/chrome-mcp/extension; \
    rm /tmp/chrome-mcp-server.zip; \
    npm install -g mcp-chrome-bridge; \
    mcp-chrome-bridge register -s -f; \
    ln -sf /usr/local/lib/node_modules/mcp-chrome-bridge /srv/mcp/chrome-mcp/package; \
    #
    # install Playwright MCP
    mkdir -p /srv/mcp/playwright-mcp; \
    wget -O /tmp/playwright-mcp-extension.zip "https://github.com/microsoft/playwright-mcp/releases/download/v0.0.35/playwright-mcp-extension-v0.0.35.zip"; \
    unzip /tmp/playwright-mcp-extension.zip -d /srv/mcp/playwright-mcp/extension; \
    chmod -R 755 /srv/mcp/playwright-mcp/extension; \
    rm /tmp/playwright-mcp-extension.zip; \
    npm install -g @playwright/mcp; \
    ln -sf /usr/local/lib/node_modules/@playwright/mcp /srv/mcp/playwright-mcp/package; \
    #
    # clean up
    apt-get --purge autoremove -y unzip wget; \
    apt-get clean -y; \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/*

#
# copy configuation files
COPY supervisord.conf /etc/neko/supervisord/chromium.conf
COPY --chown=neko preferences.json /home/neko/.config/chromium/Default/Preferences
COPY policies.json /etc/chromium/policies/managed/policies.json
COPY openbox.xml /etc/neko/openbox.xml
