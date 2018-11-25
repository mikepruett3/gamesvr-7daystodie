# escape=`

FROM lacledeslan/steamcmd:linux as seven-builder

ARG SKIP_STEAMCMD=false

# Copy cached build files (if any)
COPY --chown=SteamCMD:root  ./steamcmd-cache /output

# Download 7 Days to Die Dedicated Server via SteamCMD
RUN if [ "$SKIP_STEAMCMD" = true ] ; then `
        echo "\n\nSkipping SteamCMD install -- using only contents from steamcmd-cache\n\n"; `
    else `
        echo "\n\nDownloading 7 Days to Die Dedicated Server via SteamCMD"; `
        /app/steamcmd.sh `
            +login anonymous `
            +force_install_dir /output  `
            +app_update 294420 -beta alpha16.4 validate `
            +quit; `
    fi

#=======================================================================
FROM debian:stable-slim

ARG BUILDNODE=unspecified
ARG SOURCE_COMMIT=unspecified

HEALTHCHECK NONE

RUN dpkg --add-architecture i386 && `
    apt-get update && apt-get install -y `
        ca-certificates `
        lib32gcc1 `
        libc6-i386 `
        lib32stdc++6 `
        locales `
        locales-all `
        tmux `
        xmlstarlet `
        nano `
        wget `
        unzip && `
    apt-get clean && `
    echo "LC_ALL=en_US.UTF-8" >> /etc/environment && `
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
RUN useradd --home /app --gid root --system 7DaysToDie && `
    mkdir -p /app/ll-tests && `
    chown 7DaysToDie:root -R /app

COPY --chown=7DaysToDie:root --from=seven-builder /output /app

COPY --chown=7DaysToDie:root ./ll-tests /app/ll-tests

RUN chmod +x /app/ll-tests/*.sh

## Install Mods
#RUN mkdir -p /tmp/Mods && `
#    # StompiNZ's Bad Company Manager 
#        # Download, Unzip and Delete BCManager.zip
#        wget -P /tmp https://github.com/7days2mod/BadCompanySM/releases/download/v2.5.2_stable/BCManager.zip && `
#        unzip /tmp/BCManager.zip -d /tmp/Mods && `
#        rm -vf /tmp/BCManager.zip && `
#    # Alloc's Server Fixes
#        # Download, Unzip and Delete server_fixes_v14_17_26.tar.gz
#        wget -P /tmp http://illy.bz/fi/7dtd/server_fixes_v14_17_26.tar.gz && `
#        tar -xvzf /tmp/server_fixes_v14_17_26.tar.gz -C /tmp && `
#        rm -vf /tmp/server_fixes_v14_17_26.tar.gz && `
#    # StompiNZ's Bad Company Manager 
#        # Download, Unzip and Delete ScriptingMod_v1.0.zip
#        wget -P /tmp https://github.com/djkrose/7DTD-ScriptingMod/releases/download/v1.0/ScriptingMod_v1.0.zip && `
#        unzip /tmp/ScriptingMod_v1.0.zip -d /tmp && `
#        rm -vf /tmp/ScriptingMod_v1.0.zip

# Create /tmp/Mods
RUN mkdir -p /tmp/Mods

# Download StompiNZ's Bad Company Manager 
RUN wget -P /tmp https://github.com/7days2mod/BadCompanySM/releases/download/v2.5.2_stable/BCManager.zip
RUN unzip /tmp/BCManager.zip -d /tmp/Mods
RUN rm -vf /tmp/BCManager.zip

# Alloc's Server Fixes
RUN wget -P /tmp http://illy.bz/fi/7dtd/server_fixes_v14_17_26.tar.gz
RUN tar -xvzf /tmp/server_fixes_v14_17_26.tar.gz -C /tmp
RUN rm -vf /tmp/server_fixes_v14_17_26.tar.gz

# StompiNZ's Bad Company Manager 
RUN wget -P /tmp https://github.com/djkrose/7DTD-ScriptingMod/releases/download/v1.0/ScriptingMod_v1.0.zip
RUN unzip -j /tmp/ScriptingMod_v1.0.zip -d /tmp
RUN rm -vf /tmp/ScriptingMod_v1.0.zip

COPY --chown=7DaysToDie:root /tmp/Mods /app/Mods
    
USER 7DaysToDie

WORKDIR /app

CMD ["/bin/bash"]

ONBUILD USER root

# Volumes for Persistent Server Data
VOLUME [ "/app/.local/share", "/app/Mods"]
