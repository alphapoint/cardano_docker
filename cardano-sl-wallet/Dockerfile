FROM debian:jessie

RUN apt-get update &&\
   apt-get install -y git curl bzip2 nginx sudo nano &&\
   useradd -ms /bin/bash cardano &&\
   mkdir -m 0755 /nix &&\
   chown cardano /nix &&\
   mkdir -p /etc/nix &&\
   echo binary-caches = https://cache.nixos.org https://hydra.iohk.io > /etc/nix/nix.conf &&\
   echo binary-caches-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= >> /etc/nix/nix.conf &&\
   su - cardano -c 'git clone https://github.com/input-output-hk/cardano-sl.git /home/cardano/cardano-sl'

ADD default.conf /etc/nginx/conf.d/

ADD start-cardano-container.sh /home/cardano/cardano-sl/
RUN chmod a+x /home/cardano/cardano-sl/start-cardano-container.sh

RUN echo "cardano ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER cardano
ENV USER cardano
RUN curl https://nixos.org/nix/install | sh

WORKDIR /home/cardano/cardano-sl
RUN git checkout master &&\
   git fetch origin &&\
   git pull
RUN . /home/cardano/.nix-profile/etc/profile.d/nix.sh &&\
   nix-build -A connectScripts.mainnetWallet -o connect-to-mainnet
    
CMD /home/cardano/cardano-sl/start-cardano-container.sh
