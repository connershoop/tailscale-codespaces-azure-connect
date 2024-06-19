FROM mcr.microsoft.com/devcontainers/universal:2

RUN wget https://aka.ms/downloadazcopy-v10-linux -O - | tar -xvzf - && \
    rm -f /usr/bin/azcopy && \
    cp ./azcopy_linux_amd64_*/azcopy /usr/bin/ && \
    chmod 755 /usr/bin/azcopy && \
    rm -f downloadazcopy-v10-linux && \
    rm -rf ./azcopy_linux_amd64_*/