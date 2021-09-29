FROM debian:stretch

ENV LANG C.UTF-8

# common haskell + stack dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dirmngr \
        git \
        gcc \
        gnupg \
        g++ \
        libc6-dev \
        libffi-dev \
        libgmp-dev \
        libsqlite3-dev \
        libtinfo-dev \
        make \
        netbase \
        openssh-client \
        xz-utils \
        zlib1g-dev && \
    rm -rf /var/lib/apt/lists/*


# The cabal install process will also move away from the debian package once https://github.com/haskell/cabal/issues/7639 is resolved
ARG CABAL_INSTALL=3.4
ARG DEBIAN_KEY=427CB69AAC9D00F2A43CAF1CBA3CBA3FFE22B574

RUN export GNUPGHOME="$(mktemp -d)" && \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys ${DEBIAN_KEY} && \
    gpg --batch --armor --export ${DEBIAN_KEY} > /etc/apt/trusted.gpg.d/haskell.org.gpg.asc && \
    gpgconf --kill all && \
    echo 'deb http://downloads.haskell.org/debian stretch main' > /etc/apt/sources.list.d/ghc.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        cabal-install-${CABAL_INSTALL} && \
    rm -rf "$GNUPGHOME" /var/lib/apt/lists/*

ARG GHC=8.10.6
ARG GHC_RELEASE_KEY=88B57FCF7DB53B4DB3BFA4B1588764FBE22D19C4
ARG GHC_RELEASE_SHA256=C14B631437EBC867F1FE1648579BC1DBE1A9B9AD31D7C801C3C77639523A83AE

RUN cd /tmp && \
  export GNUPGHOME="$(mktemp -d)" && \
  curl -sSLO https://downloads.haskell.org/~ghc/$GHC/ghc-$GHC-x86_64-deb9-linux.tar.xz && \
  curl -sSLO https://downloads.haskell.org/~ghc/$GHC/ghc-$GHC-x86_64-deb9-linux.tar.xz.sig && \
  gpg --batch --keyserver keyserver.ubuntu.com --receive-keys ${GHC_RELEASE_KEY} && \
  gpg --batch --trusted-key 588764FBE22D19C4 --verify ghc-$GHC-x86_64-deb9-linux.tar.xz.sig ghc-$GHC-x86_64-deb9-linux.tar.xz && \
  tar xf ghc-$GHC-x86_64-deb9-linux.tar.xz && \
  echo "$GHC_RELEASE_SHA256 ghc-$GHC-x86_64-deb9-linux.tar.xz" | sha256sum --strict --check && \
  cd ghc-$GHC && \
  ./configure --prefix /opt/ghc/$GHC && \
  make install && \
  find /opt/ghc/$GHC/ \( -name "*_p.a" -o -name "*.p_hi" \) -type f -delete && \
  rm -rf "$GNUPGHOME" /tmp/*

# ensure any user gets GHC on the path
RUN echo "export PATH=\"/opt/ghc/$GHC/bin:\$PATH\"" >> /etc/profile.d/ghc_path.sh && \
	chmod +x /etc/profile.d/ghc_path.sh

ARG STACK=2.7.3
ARG STACK_RELEASE_KEY=C5705533DA4F78D8664B5DC0575159689BEFB442
# get from https://github.com/commercialhaskell/stack/releases/download/v${STACK}/stack-${STACK}-linux-x86_64.tar.gz.sha256
ARG STACK_RELEASE_SHA256=A6C090555FA1C64AA61C29AA4449765A51D79E870CF759CDE192937CD614E72B

RUN export GNUPGHOME="$(mktemp -d)" && \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys ${STACK_RELEASE_KEY} && \
    curl -fSL https://github.com/commercialhaskell/stack/releases/download/v${STACK}/stack-${STACK}-linux-x86_64.tar.gz -o stack.tar.gz && \
    curl -fSL https://github.com/commercialhaskell/stack/releases/download/v${STACK}/stack-${STACK}-linux-x86_64.tar.gz.asc -o stack.tar.gz.asc && \
    gpg --batch --trusted-key 0x575159689BEFB442 --verify stack.tar.gz.asc stack.tar.gz && \
    echo "$STACK_RELEASE_SHA256 stack.tar.gz" | sha256sum --strict --check && \
    tar -xf stack.tar.gz -C /usr/local/bin --strip-components=1 && \
    /usr/local/bin/stack config set system-ghc --global true && \
    /usr/local/bin/stack config set install-ghc --global false && \
    rm -rf "$GNUPGHOME" /var/lib/apt/lists/* /stack.tar.gz.asc /stack.tar.gz

ENV PATH /root/.cabal/bin:/root/.local/bin:/opt/cabal/${CABAL_INSTALL}/bin:/opt/ghc/${GHC}/bin:$PATH

CMD ["ghci"]
