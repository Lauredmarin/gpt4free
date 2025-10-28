FROM selenium/node-chrome
ARG G4F_VERSION
ENV G4F_VERSION=${G4F_VERSION:-0.1.0}
ENV SE_SCREEN_WIDTH 1850
ENV G4F_DIR /app
ENV G4F_LOGIN_URL http://localhost:7900/?autoconnect=1&resize=scale&password=secret
USER root

# If docker compose, install git
RUN if [ "$G4F_VERSION" = "0.1.0" ] ; then \
  apt-get -qqy update && \
  apt-get -qqy install git \
  ; fi

# Install Python3, pip, remove OpenJDK 11, clean up
RUN apt-get -qqy update \
  && apt-get -qqy upgrade \
  && apt-get -qyy autoremove \
  && apt-get -qqy install python3 python-is-python3 pip ffmpeg flac libavcodec-extra \
  && apt-get -qyy remove openjdk-11-jre-headless \
  && apt-get -qyy autoremove \
  && apt-get -qyy clean \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# Update entrypoint
COPY docker/supervisor.conf /etc/supervisor/conf.d/selenium.conf
COPY docker/supervisor-api.conf /etc/supervisor/conf.d/api.conf

# Change background image
COPY docker/background.png /usr/share/images/fluxbox/ubuntu-light.png

# Add user, fix permissions
RUN chown "${SEL_UID}:${SEL_GID}" $HOME/.local /opt/venv/share

# Switch user
USER $SEL_UID

# Set the working directory in the container
WORKDIR $G4F_DIR

# Copy requirements and install dependencies
COPY --chown=$SEL_UID:$SEL_GID requirements.txt $G4F_DIR/
RUN pip install --break-system-packages --upgrade pip \
  && pip install --break-system-packages -r requirements.txt

# Copy the entire project (setup.py + g4f + README.md)
COPY --chown=$SEL_UID:$SEL_GID setup.py $G4F_DIR/
COPY --chown=$SEL_UID:$SEL_GID README.md $G4F_DIR/
COPY --chown=$SEL_UID:$SEL_GID g4f $G4F_DIR/g4f

# Install g4f package in editable mode
RUN pip install --break-system-packages -e .

# Expose ports
EXPOSE 8080 7900
