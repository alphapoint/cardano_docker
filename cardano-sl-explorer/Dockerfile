FROM debian:jessie

RUN apt-get update &&\
    apt-get install -y git curl bzip2 &&\
    useradd -ms /bin/bash cardano &&\
    mkdir -m 0755 /nix &&\
    chown cardano /nix &&\
    mkdir -p /etc/nix &&\
    echo binary-caches = https://cache.nixos.org https://hydra.iohk.io > /etc/nix/nix.conf &&\
    echo binary-cache-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY= >> /etc/nix/nix.conf &&\
    su - cardano -c 'git clone https://github.com/input-output-hk/cardano-sl.git /home/cardano/cardano-sl'

USER cardano
ENV USER cardano
RUN curl https://nixos.org/nix/install | sh

WORKDIR /home/cardano/cardano-sl
RUN git checkout tags/1.1.0
RUN . /home/cardano/.nix-profile/etc/profile.d/nix.sh &&\
    nix-build -A cardano-sl-explorer-static -o cardano-sl-explorer-static
RUN . /home/cardano/.nix-profile/etc/profile.d/nix.sh &&\
    nix-build release.nix -A connectScripts.mainnetExplorer -o connect-explorer-to-mainnet

CMD ./connect-explorer-to-mainnet
