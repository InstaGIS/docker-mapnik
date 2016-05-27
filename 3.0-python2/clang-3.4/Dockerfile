FROM ubuntu:14.04.4

# Python 2 is used for the compilation of Mapnik, and the Python bindings are
#	compiled and installed for Python 2.

# Install necessary packages.
# note: if the following warning is printed, do not worry.
#   "debconf: delaying package configuration, since apt-utils is not installed"
# warning: do NOT split this in more lines; it has to be a single RUN.
RUN apt-get update -q && apt-get install -y \
	make curl wget unzip \
	git \
	clang \
	build-essential \
	pkg-config \
	python2.7 \
	python2.7-dev \
	gdal-bin \
	zlib1g-dev \
	libproj-dev \
	libboost-python-dev \
	libboost-thread-dev

# Make sure Python outputs everything that's printed inside the application
#   rather than buffering it.
ENV PYTHONUNBUFFERED 1

# install 'pip' and 'setuptools' for Python 2
RUN curl -sL 'https://bootstrap.pypa.io/get-pip.py' | python2 \
	&& pip install --upgrade setuptools

RUN git clone https://github.com/mapnik/mapnik mapnik-3.x --depth 10 \
	&& cd mapnik-3.x \
	&& git submodule update --init

# note: had to split in 3 'RUN's because 'source' is a bash function that is not
#	available in 'sh'.
RUN ["/bin/bash", "-c", "cd mapnik-3.x && source bootstrap.sh"]

RUN cd mapnik-3.x \
	&& PYTHON=python2 ./configure CUSTOM_CXXFLAGS="-D_GLIBCXX_USE_CXX11_ABI=0" \
	&& make PYTHON=python2 \
	&& make install

# note: '--no-verbose' is less aggressive than 'quiet' which supresses all output
RUN wget --no-verbose 'https://github.com/mapnik/python-mapnik/archive/e2bbfa6.zip' \
	&& mv e2bbfa6.zip python-mapnik-e2bbfa6.zip \
	&& unzip -q python-mapnik-e2bbfa6.zip \
	&& mv python-mapnik-e2bbfa6e21cbe109e23a3dc28eb12898e28741c2 python-mapnik-e2bbfa6

# note: had to split in 3 'RUN's because 'source' is a bash function that is not
#	available in 'sh'.
RUN ["/bin/bash", "-c", "cd python-mapnik-e2bbfa6 && source bootstrap.sh"]

# note: we need to clean up before running post bindings-installation check.
RUN cd python-mapnik-e2bbfa6/ \
	&& BOOST_PYTHON_LIB="boost_python-py27" BOOST_SYSTEM_LIB="boost_system" BOOST_THREAD_LIB="boost_thread" \
		python2 setup.py install \
	&& cd .. \
	&& rm -R python-mapnik-e2bbfa6/ \
	&& python2 -c "import mapnik; print(mapnik.__file__)"
# output should be something like:
#	'/usr/local/lib/python2.7/dist-packages/mapnik-0.1-py2.7-linux-x86_64.egg/mapnik/__init__.pyc'

# note: logs are sent to stdout and stderr.
# note: the array form and executable full path is the preferred format of CMD,
#    and it also means the command will be run without a shell.
# warning: do not use 'ENTRYPOINT' because we want to be able to override the
#   default command by passing arguments to 'docker run'.
CMD ["mapnik-config", "-v"]
