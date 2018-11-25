
FROM lacledeslan/steamcmd:linux as seven-builder

ARG SKIP_STEAMCMD=false

# Copy cached build files (if any)
COPY --chown=SteamCMD:root  ./steamcmd-cache /output

# Download 7 Days to Die Dedicated Server via SteamCMD
RUN if [ "$SKIP_STEAMCMD" = true ] ; then \
        echo "\n\nSkipping SteamCMD install -- using only contents from steamcmd-cache\n\n"; \
    else \
        echo "\n\nDownloading 7 Days to Die Dedicated Server via SteamCMD"; \
        /app/steamcmd.sh \
            +login anonymous \
            +force_install_dir /output \
            +app_update 294420 -beta alpha16.4 validate \
            +quit; \
    fi

#=======================================================================
FROM debian:stable-slim

ARG BUILDNODE=unspecified
ARG SOURCE_COMMIT=unspecified

HEALTHCHECK NONE

RUN dpkg --add-architecture i386 && \
    apt-get update && apt-get install -y \
        ca-certificates \
        lib32gcc1 \
        libc6-i386 \
        lib32stdc++6 \
        locales \
        locales-all \
        tmux \
        xmlstarlet \
        nano \
        wget \
        unzip && \
    apt-get clean && \
    echo "LC_ALL=en_US.UTF-8" >> /etc/environment && \
    rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

LABEL com.lacledeslan.build-node=$BUILDNODE
LABEL org.label-schema.schema-version="1.0"
LABEL org.label-schema.url="https://github.com/LacledesLAN/README.1ST"
LABEL org.label-schema.vcs-ref=$SOURCE_COMMIT
LABEL org.label-schema.vendor="Laclede's LAN"
LABEL org.label-schema.description="7 Days to Die Dedicated Server"
LABEL org.label-schema.vcs-url="https://github.com/LacledesLAN/gamesvr-7daystodie"

# Set up Enviornment
RUN useradd --home /app --gid root --system 7DaysToDie && \
    mkdir -p /app/ll-tests && \
    chown 7DaysToDie:root -R /app

COPY --chown=7DaysToDie:root --from=seven-builder /output /app

COPY --chown=7DaysToDie:root ./ll-tests /app/ll-tests

RUN chmod +x /app/ll-tests/*.sh

# Download and Prepair Mods
RUN \
    mkdir /app/Mods && \
    echo "Downloading StompiNZ's Bad Company Manager" && \
    wget -P /app/Mods https://github.com/7days2mod/BadCompanySM/releases/download/v2.5.2_stable/BCManager.zip && \
    echo "Unzipping BCManager.zip to /app/Mods" && \
    unzip /app/Mods/BCManager.zip -d /app/Mods && \
    echo "Deleting BCManager.zip from /app/Mods" && \
    rm -vf /app/Mods/BCManager.zip && \
    echo "Downloading Alloc's Server Fixes" && \
    wget -P /app/Mods http://illy.bz/fi/7dtd/server_fixes_v14_17_26.tar.gz && \
    echo "Untaring server_fixes_v14_17_26.tar.gz to /app/Mods" && \
    tar -xvzf /app/Mods/server_fixes_v14_17_26.tar.gz -C /app && \
    echo "Deleting server_fixes_v14_17_26.tar.gz from /app/Mods" && \
    rm -vf /app/Mods/server_fixes_v14_17_26.tar.gz && \
    echo "Downloading djkrose's Scripting Mod" && \
    wget -P /app/Mods https://github.com/djkrose/7DTD-ScriptingMod/releases/download/v1.0/ScriptingMod_v1.0.zip && \
    echo "Unzipping ScriptingMod_v1.0.zip to /app/Mods" && \
    unzip /app/Mods/ScriptingMod_v1.0.zip -d /app; exit 0 && \
    echo "Deleting ScriptingMod_v1.0.zip from /app/Mods" && \
    rm -vf /app/Mods/ScriptingMod_v1.0.zip && \
    sleep 15s && \
    echo "Change Owner of /app/Mods to 7DaysToDie" && \
    chown --recursive 999:root /app/Mods && \
    echo "Change Owner of /app/.local to 7DaysToDie" && \
    chown --recursive 999:root /app/.local/share/7DaysToDie

USER 7DaysToDie

WORKDIR /app

CMD ["/bin/bash"]

ONBUILD USER root

# Volumes for Persistent Server Data
VOLUME [ "/app/.local/share/7DaysToDie/Saves", "/app/Mods" ]
