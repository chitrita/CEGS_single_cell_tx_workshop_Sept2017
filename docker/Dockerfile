FROM ubuntu:16.04
MAINTAINER bhaas@broadinstitute.org

RUN apt-get update && apt-get install -y gcc g++ perl python automake make cmake less vim \
                                       wget git curl libdb-dev \
                                       zlib1g-dev bzip2 libncurses5-dev \
                                       texlive-latex-base \
                                       default-jre \
                                       python-pip python-dev \
                                       gfortran \
                                       build-essential libghc-zlib-dev libncurses-dev libbz2-dev liblzma-dev libpcre3-dev libxml2-dev \
                                       libblas-dev gfortran git unzip ftp libzmq3-dev nano ftp fort77 libreadline-dev \
                                       libcurl4-openssl-dev libx11-dev libxt-dev \
                                       x11-common libcairo2-dev libpng12-dev libreadline6-dev libjpeg8-dev pkg-config \
                                       build-essential cmake gsl-bin libgsl0-dev libeigen3-dev libboost-all-dev libssl-dev libssl-dev libcairo2-dev libxt-dev libgtk2.0-dev libcairo2-dev xvfb xauth xfonts-base

RUN apt-get install -y apt-transport-https

## Install R-3.4.1
COPY installR.sh /usr/local/src

RUN /usr/local/src/installR.sh


RUN apt-get install -y sudo locales

RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
  && locale-gen en_US.utf8 \
    && /usr/sbin/update-locale LANG=en_US.UTF-8

RUN apt-get install -y gdebi-core

COPY install_rstudio_server.sh /usr/local/src

RUN /usr/local/src/install_rstudio_server.sh

### User setup
RUN groupadd -g 2000 training && useradd -m -u 2000 -g 2000 training
RUN echo 'training:training' | chpasswd
RUN chsh -s /bin/bash training
ENV HOME=/home/training
RUN echo "alias ll='ls -la -G'" >> /home/training/.profile
RUN usermod -G training,www-data training


RUN apt-get install -y openssh-server libncurses5-dev apache2 supervisor

ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf
CMD ["/usr/bin/supervisord","-c","/etc/supervisor/conf.d/supervisord.conf"]


#########
### GateOne SSH interface
#########
RUN git clone https://github.com/liftoff/GateOne/ && \
    cd GateOne && python setup.py install && \
     python run_gateone.py --configure && cd ..



EXPOSE 22 80 443 8787


##############
## Pagoda

RUN wget https://github.com/searchivarius/nmslib/archive/v1.5.3.tar.gz && \
	 tar xvzf v1.5.3.tar.gz && \
	 cd nmslib-1.5.3 && \
	 cd similarity_search && \
	 cmake . && \
	 make && \
	 make install


RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("devtools", dep = TRUE)'
RUN wget http://pklab.med.harvard.edu/pagoda2dependencies/largeVis.tar.gz && \
        tar xvzf largeVis.tar.gz && \
        Rscript -e 'library(devtools); install("largeVis");'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("tidyverse", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite(c("GO.db", "org.Hs.eg.db", "org.Mm.eg.db", "pcaMethods"), dep = TRUE)'
#RUN Rscript -e 'library(devtools); install_github("igraph/rigraph");'



RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("Cairo", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("urltools", dep = TRUE)'



RUN Rscript -e 'library(devtools); install_github("RGLab/Rtsne.multicore")'

RUN cp -r /nmslib-1.5.3/similarity_search/lshkit/include/* /usr/local/include/

RUN apt-get install -y software-properties-common && \
    add-apt-repository -y "ppa:marutter/rrutter" && \
    add-apt-repository -y "ppa:marutter/c2d4u" && \
    apt-get update && apt-get install -y r-cran-igraph && \
    Rscript -e 'library(devtools); install_github("hms-dbmi/pagoda2")'


