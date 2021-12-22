ARG DEBIAN_VERSION=buster-slim

FROM debian:${DEBIAN_VERSION} AS chktex
ARG CHKTEX_VERSION=1.7.6
WORKDIR /tmp/workdir
RUN apt-get update -y && \
  apt-get install -y --no-install-recommends g++ make wget
RUN wget -qO- http://download.savannah.gnu.org/releases/chktex/chktex-${CHKTEX_VERSION}.tar.gz | \
  tar -xz --strip-components=1
RUN ./configure && \
  make && \
  mv chktex /tmp && \
  rm -r *

FROM qmcgaw/basedevcontainer:debian
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=1000
USER root
RUN apt-get update -y && \
  apt-get install -y --no-install-recommends texlive texlive-latex-extra texlive-extra-utils texlive-fonts-extra texlive-bibtex-extra texlive-lang-german texlive-fonts-recommended texlive-publishers texlive-science && \
  # Cleanup
  cd && \
  apt-get clean autoclean && \
  apt-get autoremove -y && \
  rm -rf /var/lib/{apt,dpkg,cache,log}/ /tmp/texlive 
WORKDIR /workspace
# Latexindent dependencies
RUN apt-get update -y && \
  apt-get install -y --no-install-recommends cpanminus make gcc libc6-dev && \
  cpanm -n -q Log::Log4perl && \
  cpanm -n -q XString && \
  cpanm -n -q Log::Dispatch::File && \
  cpanm -n -q YAML::Tiny && \
  cpanm -n -q File::HomeDir && \
  cpanm -n -q Unicode::GCString && \
  apt-get remove -y cpanminus make gcc libc6-dev && \
  apt-get clean autoclean && \
  apt-get autoremove -y && \
  rm -rf /var/lib/{apt,dpkg,cache,log}/
# Add vscode user
#RUN groupadd --gid $USER_GID $USERNAME \
#  && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME
#
# [Optional] Add sudo support. Omit if you don't need to install software after connecting.
# && apt-get update \
#&& apt-get install -y sudo \
#&& echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
#&& chmod 0440 /etc/sudoers.d/$USERNAME
USER ${USERNAME}
RUN tlmgr init-usertree
RUN tlmgr install latexindent latexmk && \
  texhash
COPY --from=chktex --chown=${USER_UID}:${USER_GID} /tmp/chktex /usr/local/bin/chktex
COPY --chown=${USER_UID}:${USER_GID} shell/.zshrc-specific shell/.welcome.sh /home/${USERNAME}/
COPY shell/.zshrc-specific shell/.welcome.sh /root/
# Verify binaries work and have the right permissions
RUN tlmgr version && \
  latexmk -version && \
  texhash --version && \
  chktex --version
