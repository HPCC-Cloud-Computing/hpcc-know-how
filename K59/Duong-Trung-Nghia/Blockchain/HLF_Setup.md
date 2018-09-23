# Hyperledger Fabric

## Prerequisites

1. CURL la gi?
    * https://www.cyberciti.biz/faq/how-to-install-curl-command-on-a-ubuntu-linux/

1. Docker & Docker Compose
    * https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/
    * https://docs.docker.com/compose/install

1. GoLang 
    * https://medium.com/@patdhlk/how-to-install-go-1-7-4-on-ubuntu-16-04-ee64c073cd79
    * https://www.itzgeek.com/how-tos/linux/centos-how-tos/install-go-1-7-ubuntu-16-04-14-04-centos-7-fedora-24.html
    * https://golang.org/doc/install?download=go1.9.3.darwin-amd64.pkg
    * Setup Go environment: 
        * Ubuntu
        ```ubuntu
        GOPATH=$HOME/.go
        export GOPATH
        PATH=$PATH:$GOPATH/bin # Add GOPATH/bin to PATH for scripting
        ```
        * MacOS
        ```MacOS
        export GOPATH=$HOME//go
        export PATH=$GOPATH/bin:$PATH
        ```

        ```sh 
        mv /usr/local/bin/libtool /usr/local/bin/libtool.bak
        npm install scrypt
        mv /usr/local/bin/libtool.bak /usr/local/bin/libtool
        ```

        ```sh
        sudo apt-get install make
        sudo apt-get install g++
        sudo npm install node-gyp -g
        ```

1. NodeJS & npm
    * https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions
    ```sh
    curl "https://nodejs.org/dist/v8.9.3/node-${VERSION:-$(wget -qO- https://nodejs.org/dist/v8.9.3/ | sed -nE 's|.*>node-(.*)\.pkg</a>.*|\1|p')}.pkg" > "$HOME/Downloads/node-latest.pkg" && sudo installer -store -pkg "$HOME/Downloads/node-latest.pkg" -target "/"
    ```

1. Nginx
    
    * Build-essential
    ```sh
    apt-get install build-essential
    ```
    * PCRE – Supports regular expressions
    ```sh
    wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.41.tar.gz
    tar -zxf pcre-8.41.tar.gz
    cd pcre-8.41
    ./configure
    make
    sudo make install
    ```
    * zlib – Supports header compression
    ```sh
    wget http://zlib.net/zlib-1.2.11.tar.gz
    tar -zxf zlib-1.2.11.tar.gz
    cd zlib-1.2.11
    ./configure
    make
    sudo make install
    ```
    * OpenSSL – Supports the HTTPS protocol.
    ```sh
    wget http://www.openssl.org/source/openssl-1.0.2k.tar.gz
    tar -zxf openssl-1.0.2k.tar.gz
    cd openssl-1.0.2k
    ./config # See platform are using. See list platforms: ./Configure LIST 
    ./Configure linux-x86_64 --prefix=/usr
    make
    sudo make install
    ```

    * Nginx

        ```sh 
        wget http://nginx.org/download/nginx-1.12.1.tar.gz
        tar -zxf nginx-1.12.1.tar.gz
        cd nginx-1.12.1
        ```
        * Choosing statis streams
        ```sh
        ./configure \
        --sbin-path=/usr/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --pid-path=/var/run/nginx.pid \
        --modules-path=/usr/lib/nginx/modules \
        --with-pcre=../pcre-8.41 --with-zlib=../zlib-1.2.11 \
        --with-http_ssl_module --with-stream --with-mail
        ```
        * Choosing dymamic stream, mail modules
        ```sh
        ./configure \
        --sbin-path=/usr/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --pid-path=/var/run/nginx.pid \
        --modules-path=/usr/lib/nginx/modules \
        --with-pcre=../pcre-8.41 --with-zlib=../zlib-1.2.11 \
        --with-http_ssl_module \
        --with-mail=dynamic \
        --with-stream=dynamic 
        ```

        ```sh
        make
        sudo make install
        ```
## Các khái niệm cơ bản

### What is a Blockchain?

1. A Distributed Ledger

1. Smart Contract

1. Consensus (Sự đồng thuận) 

### Why is a Blockchain useful?

### What is Hyperledger Fabric?

* Membership Service Provider (MSP)

* Shared Ledger: world state & transaction log

* Smart Contracts: chaincode

* Privacy

* Consensus

## Hyperledger Fabric Capabilities

## Hyperledger Fabric Model

## References

* https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
