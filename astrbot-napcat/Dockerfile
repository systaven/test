# ==============================================================================
# 阶段 1: 准备 NapCat 环境 (原始镜像文件源)
# ==============================================================================
FROM mlikiowa/napcat-docker:latest AS napcat_build

# ==============================================================================
# 阶段 2: 准备 AstrBot 环境 (原始镜像文件源)
# ==============================================================================
FROM soulter/astrbot:latest AS astrbot_build

# ==============================================================================
# 阶段 3: 最终构建 - 融合和配置 (使用轻量级 Python 基础镜像)
# ==============================================================================
FROM python:3.10-slim

# 1. 安装系统工具 (Supervisor, 权限管理, NTQQ 系统依赖, DBus, Xvfb)
# 修复了所有已知的系统环境和库缺失问题
RUN apt-get update && \
    apt-get install -y supervisor bash python3-pip unzip gosu passwd dbus xvfb \
    libglib2.0-0 libgtk-3-0 libnss3 libasound2 libsecret-1-0 libfontconfig1 \
    libgbm1 libdrm2 libxkbcommon0 \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 2. 手动创建 NapCat 用户/组
RUN groupadd --gid 1000 napcat || true && useradd -m -s /bin/bash -u 1000 -g napcat napcat || true

# 3. 复制 NapCat 的核心文件和 QQ 本体
COPY --from=mlikiowa/napcat-docker:latest /app /app
COPY --from=mlikiowa/napcat-docker:latest /opt/QQ /opt/QQ

# 4. 修复 NapCat 的 Python 依赖 (Poetry)
RUN /usr/local/bin/python3 -m pip install poetry --break-system-packages

# 5. 复制 AstrBot 的核心文件
COPY --from=soulter/astrbot:latest /AstrBot /AstrBot

# 6. 安装 AstrBot 缺失的依赖 (完整列表)
RUN /usr/local/bin/python3 -m pip install \
    aiohttp pydantic~=2.10.3 psutil>=5.8.0 openai anthropic qq-botpy chardet~=5.1.0 Pillow beautifulsoup4 \
    mi-googlesearch-python readability-lxml quart lxml_html_clean colorlog aiocqhttp pyjwt apscheduler \
    docstring_parser aiodocker silk-python lark-oapi ormsgpack cryptography dashscope python-telegram-bot \
    wechatpy dingtalk-stream defusedxml mcp certifi pip telegramify-markdown google-genai click filelock \
    watchfiles websockets faiss-cpu aiosqlite py-cord>=2.6.1 slack-sdk pydub sqlmodel deprecated \
    'sqlalchemy[asyncio]' --break-system-packages

# 7. 复制预配置好的 AstrBot 配置
COPY astrbot_config_template/ /AstrBot/data/

# 8. 配置 Supervisord
COPY supervisord.conf /etc/supervisor/supervisord.conf

# 9. 设置工作目录和环境变量
WORKDIR /
ENV TZ=Asia/Shanghai

# 10. 暴露端口
EXPOSE 6099 6185

# 11. 启动 Supervisord
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]