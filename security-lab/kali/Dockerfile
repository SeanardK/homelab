FROM kalilinux/kali-rolling

ENV DEBIAN_FRONTEND=noninteractive

# kali-linux-headless: the official metapackage for non-GUI hosts — every
# tool from kali-linux-default minus anything that needs a desktop
# (Burp Suite GUI, GUI Wireshark, etc). This is a large install (~6-10GB).
RUN apt-get update && \
    apt-get install -y --no-install-recommends kali-linux-headless && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /root
CMD ["sleep", "infinity"]
