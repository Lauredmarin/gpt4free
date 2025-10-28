FROM selenium/node-chrome
ARG G4F_VERSION
ENV G4F_VERSION $G4F_VERSION
ENV SE_SCREEN_WIDTH 1850
ENV G4F_DIR /app
ENV G4F_LOGIN_URL http://localhost:7900/?autoconnect=1&resize=scale&password=secret
USER root

# If docker compose, install git
RUN if [ "$G4F_VERSION" = "" ] ; then \
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

# Copy the entire project first (including setup.py if it exists)
COPY --chown=$SEL_UID:$SEL_GID . $G4F_DIR

# Upgrade pip and install requirements
RUN pip install --break-system-packages --upgrade pip \
  && pip install --break-system-packages -r requirements.txt

# Install g4f as a package (if setup.py exists)
# Option 1: If you have a setup.py file
RUN if [ -f setup.py ]; then pip install --break-system-packages -e . ; fi

# Option 2: If no setup.py, install in development mode by adding to PYTHONPATH
ENV PYTHONPATH="${PYTHONPATH}:${G4F_DIR}"

# Expose ports
EXPOSE 8080 7900
