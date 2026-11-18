# 基础镜像：Node.js 22（与 devcontainer.json 的 build.args 保持一致）
ARG NODE_VERSION=22
FROM mcr.microsoft.com/devcontainers/javascript-node:${NODE_VERSION}

# ---------------------------------------------------------------------------
# 为什么改成下面这种写法（原配置导致 Codespaces 反复进入 recovery mode）：
#
# 1) 原脚本用 api.github.com/repos/gohugoio/hugo/releases/latest 查询最新版本，
#    Codespaces 构建机的出口 IP 是共享的，极易触发 GitHub API 限流，
#    返回的是错误 JSON 而不是 tag_name，于是 VERSION 为空，
#    后面 wget 的地址变成 .../download/v/... 直接 404，镜像构建失败。
# 2) Hugo v0.170.0 起官方已经不再提供 extended 版本（hugo_extended_*），
#    所以即使 API 没被限流，"latest + extended" 也一定会 404。
#
# 结论：不要把「联网查版本」放在镜像构建里，改为固定版本号 + 固定资产名，
# 构建结果可复现；需要升级时只改下面的 ARG（或 devcontainer.json 中的 build.args）。
# ---------------------------------------------------------------------------

# Hugo 版本：0.154.2 属于仍提供 extended 构建的版本（与 .github/workflows 中的 HUGO_VERSION 一致）
ARG HUGO_VERSION=0.154.2
# Dart Sass 版本：Stack 主题的 SCSS 需要它（与 .github/workflows 中的 DART_SASS_VERSION 一致）
ARG DART_SASS_VERSION=1.97.1

# 安装 Hugo extended
# 发布资产命名规则：hugo_extended_<版本>_linux-amd64.tar.gz / linux-arm64.tar.gz
RUN set -eux; \
    ARCH="$(dpkg --print-architecture)"; \
    case "${ARCH}" in \
      amd64) HUGO_ARCH=amd64 ;; \
      arm64) HUGO_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: ${ARCH}" >&2; exit 1 ;; \
    esac; \
    curl -fsSL -o /tmp/hugo.tar.gz "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-${HUGO_ARCH}.tar.gz"; \
    tar -xzf /tmp/hugo.tar.gz -C /usr/local/bin hugo; \
    rm -f /tmp/hugo.tar.gz; \
    hugo version

# 安装 Dart Sass
# 发布资产命名规则：dart-sass-<版本>-linux-x64.tar.gz / linux-arm64.tar.gz
RUN set -eux; \
    ARCH="$(dpkg --print-architecture)"; \
    case "${ARCH}" in \
      amd64) SASS_ARCH=x64 ;; \
      arm64) SASS_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: ${ARCH}" >&2; exit 1 ;; \
    esac; \
    curl -fsSL -o /tmp/dart-sass.tar.gz "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-${SASS_ARCH}.tar.gz"; \
    mkdir -p /opt/dart-sass; \
    tar -xzf /tmp/dart-sass.tar.gz -C /opt/dart-sass --strip-components=1; \
    rm -f /tmp/dart-sass.tar.gz; \
    ln -sf /opt/dart-sass/sass /usr/local/bin/sass; \
    sass --version

# Hugo 本地预览端口
EXPOSE 1313
