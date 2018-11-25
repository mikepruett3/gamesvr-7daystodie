
FROM lacledeslan/steamcmd:linux as seven-builder

ARG SKIP_STEAMCMD=false

# Copy cached build files (if any)
COPY --chown=SteamCMD:root  ./steamcmd-cache /output

# Download 7 Days to Die Dedicated Server via SteamCMD
RUN \
    if [ "$SKIP_STEAMCMD" = true ] ; then \
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

RUN \
    dpkg --add-architecture i386 && \
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
        unzip \
        p7zip-full && \
    apt-get clean && \
    echo "LC_ALL=en_US.UTF-8" >> /etc/environment && \
    rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

LABEL com.lacledeslan.build-node=$BUILDNODE \
    org.label-schema.schema-version="1.0" \
    org.label-schema.url="https://github.com/LacledesLAN/README.1ST" \
    org.label-schema.vcs-ref=$SOURCE_COMMIT \
    org.label-schema.vendor="Laclede's LAN" \
    org.label-schema.description="7 Days to Die Dedicated Server" \
    org.label-schema.vcs-url="https://github.com/LacledesLAN/gamesvr-7daystodie"

# Set up Enviornment
RUN \
    useradd --home /app --gid root --system 7DaysToDie && \
    mkdir -p /app/ll-tests && \
    chown 7DaysToDie:root -R /app

COPY --chown=7DaysToDie:root --from=seven-builder /output /app

COPY --chown=7DaysToDie:root ./ll-tests /app/ll-tests

RUN chmod +x /app/ll-tests/*.sh

# Download and Prepair Mods
RUN \
    mkdir /app/Mods && \
    echo "Installing StompiNZ's Bad Company Manager" && \
    wget -P /app/Mods https://github.com/7days2mod/BadCompanySM/releases/download/v2.5.2_stable/BCManager.zip && \
    unzip /app/Mods/BCManager.zip -d /app/Mods && \
    rm -vf /app/Mods/BCManager.zip && \
    echo "Installing Alloc's Server Fixes" && \
    wget -P /app/Mods http://illy.bz/fi/7dtd/server_fixes_v14_17_26.tar.gz && \
    tar -xvzf /app/Mods/server_fixes_v14_17_26.tar.gz -C /app && \
    rm -vf /app/Mods/server_fixes_v14_17_26.tar.gz && \
    echo "Installing djkrose's Scripting Mod" && \
    mkdir /app/zipfix &&\
    wget -P /app/zipfix https://github.com/djkrose/7DTD-ScriptingMod/releases/download/v1.0/ScriptingMod_v1.0.zip && \
    7z rn /app/zipfix/ScriptingMod_v1.0.zip $(7z l /app/zipfix/ScriptingMod_v1.0.zip | grep '\\' | awk '{ print $6, gsub(/\\/, "/",$0); print $6}' | awk '{ print $1}' | paste -s) && \
    7z x /app/zipfix/ScriptingMod_v1.0.zip -o/app && \
    rm -vrf /app/zipfix && \
    echo "Change Owner of /app/Mods to 7DaysToDie" && \
    chown --recursive 999:root /app/Mods && \
    echo "Change Owner of /app/Saves to 7DaysToDie" && \
    chown --recursive 999:root /app/Saves && \
    echo "Modifying serverconfig.xml" && \
    sed 's/<!--property name="SaveGameFolder"/<property name="SaveGameFolder"/g' /app/serverconfig.xml && \
    sed 's/value="absolute path" \/-->/value="\/app\/Saves"\/>/g' /app/serverconfig.xml
    echo "Change Owner of //app/serverconfig.xml to 7DaysToDie" && \
    chown 999:root /app/serverconfig.xml && \

USER 7DaysToDie

WORKDIR /app

CMD ["/bin/bash"]

ONBUILD USER root

# Volumes for Persistent Server Data
VOLUME [ "/app/Saves", "/app/Mods" ]
