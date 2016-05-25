FROM ubuntu:14.04.4

# Install necessary packages.
#	'libboost-program-options-dev' is the only one that is optional
# note: if the following warning is printed, do not worry.
#   "debconf: delaying package configuration, since apt-utils is not installed"
# warning: do NOT split this in more lines; it has to be a single RUN.
RUN apt-get update -q && apt-get install -y \
	wget make python \
	build-essential \
	python2.7-dev \
	pkg-config \
	libfreetype6-dev \
	zlib1g-dev \
	libpng12-dev \
	libxml2-dev \
	libicu-dev \
	libjpeg-dev \
	libproj-dev \
	libtiff5-dev \
	libboost-dev \
	libboost-python-dev \
	libboost-system-dev \
	libboost-filesystem-dev \
	libboost-regex-dev \
	libboost-thread-dev \
	libboost-program-options-dev

# Make sure Python outputs everything that's printed inside the application
#   rather than buffering it.
ENV PYTHONUNBUFFERED 1

# note: '--no-verbose' is less aggressive than 'quiet' which supresses all output
RUN wget --no-verbose 'http://mapnik.s3.amazonaws.com/dist/v2.2.0/mapnik-v2.2.0.tar.bz2' \
	&& tar xf mapnik-v2.2.0.tar.bz2 \
	&& cd mapnik-v2.2.0/ \ 
	&& ./configure \
	&& make \
	&& make install

# note: logs are sent to stdout and stderr.
# note: the array form and executable full path is the preferred format of CMD,
#    and it also means the command will be run without a shell.
# warning: do not use 'ENTRYPOINT' because we want to be able to override the
#   default command by passing arguments to 'docker run'.
CMD ["mapnik-config", "-v"]
