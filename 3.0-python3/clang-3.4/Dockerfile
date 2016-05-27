FROM ubuntu:14.04.4

# Image for:
#	- 'mapnik/mapnik' @ master.
#	- 'mapnik/python-mapnik' @ rev e2bbfa6 (2016.05.24).

# Compilers:
#	- C: clang-3.4
#	- C++: clang-3.4

# Python 2 is used for the compilation of Mapnik but the Python bindings are
#	compiled and installed for Python 3.

# TODO: complete list.
# Required packages:
#	- ca-certificates: for 'curl', 'git' and similar using https.
#	- openssl: for 'curl', 'git' and similar using https.
#	- curl: to get 'pip'.
#	- git: besides explicit calls in this file, it is used in 'bootstrap.sh'.
#	- clang-3.4: provides 'clang++-3.4'.
#	- python: used in 'bootstrap.sh' and './configure'.
#	- zlib1g-dev: it is used in 'bootstrap.sh' and for compilation.

# Install necessary packages.
# note: if the following warning is printed, do not worry.
#   "debconf: delaying package configuration, since apt-utils is not installed"
# warning: do NOT split this in more lines; it has to be a single RUN.
RUN apt-get update -q && apt-get install -y --no-install-recommends \
	make unzip \
	ca-certificates openssl \
	curl wget \
	git \
	clang clang-3.4 \
	build-essential \
	pkg-config \
	python \
	python2.7 \
	python3.4 \
	python3.4-dev \
	gdal-bin \
	zlib1g-dev \
	libproj-dev \
	libboost-python-dev \
	libboost-thread-dev

# Make sure Python outputs everything that's printed inside the application
#   rather than buffering it.
ENV PYTHONUNBUFFERED 1

# install 'pip' and 'setuptools' for Python 3
RUN curl -sL 'https://bootstrap.pypa.io/get-pip.py' | python3 \
	&& pip install --upgrade setuptools

RUN git clone --depth=10 --branch=master 'https://github.com/mapnik/mapnik.git' mapnik \
	&& cd mapnik \
	&& git submodule update --init

# warning: we may not specify 'CXX' and 'CC' as we do for 'clang-3.5' because
#	it causes an error on './configure':
#	"C++ compiler does not support C++11 standard (-std=c++11), which is
#	required. Please upgrade your compiler to at least g++ 4.7 (ideally 4.8)".
#ENV CXX="clang++-3.4 -Qunused-arguments" CC="clang-3.4"

# note: had to split in 3 'RUN's because 'source' is a bash function that is not
#	available in 'sh'.
RUN ["/bin/bash", "-c", "cd mapnik && source bootstrap.sh"]

RUN cd mapnik \
	&& PYTHON=python2 ./configure CUSTOM_CXXFLAGS="-D_GLIBCXX_USE_CXX11_ABI=0" \
	&& make PYTHON=python2 \
	&& make install

# note: '--no-verbose' is less aggressive than 'quiet' which supresses all output
RUN wget --no-verbose 'https://github.com/mapnik/python-mapnik/archive/e2bbfa6.zip' \
	&& unzip -q e2bbfa6.zip \
	&& rm e2bbfa6.zip \
	&& mv python-mapnik-e2bbfa6e21cbe109e23a3dc28eb12898e28741c2 python-mapnik

# note: had to split in 3 'RUN's because 'source' is a bash function that is not
#	available in 'sh'.
RUN ["/bin/bash", "-c", "cd python-mapnik && source bootstrap.sh"]

# note: we need to clean up before running post bindings-installation check.
RUN cd python-mapnik/ \
	&& BOOST_PYTHON_LIB="boost_python-py34" BOOST_SYSTEM_LIB="boost_system" BOOST_THREAD_LIB="boost_thread" \
		python3 setup.py install \
	&& cd .. \
	&& rm -R python-mapnik/ \
	&& python3 -c "import mapnik; print(mapnik.__file__)"
# output should be something like:
#	'/usr/local/lib/python3.4/dist-packages/mapnik-0.1-py3.4-linux-x86_64.egg/mapnik/__init__.py'

# note: logs are sent to stdout and stderr.
# note: the array form and executable full path is the preferred format of CMD,
#    and it also means the command will be run without a shell.
# warning: do not use 'ENTRYPOINT' because we want to be able to override the
#   default command by passing arguments to 'docker run'.
CMD ["mapnik-config", "-v"]
