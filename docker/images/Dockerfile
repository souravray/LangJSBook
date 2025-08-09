# Stage 1: Build ZeroMQ (libzmq) from source
# This stage creates the ZeroMQ messaging library required for Jupyter communication
FROM debian:bullseye AS zmq-builder
# Install essential build dependencies and git for source fetching
RUN apt-get update && apt-get install -y --no-install-recommends \
	git \
	cmake \
	build-essential \
	ca-certificates \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*
WORKDIR /
RUN git clone https://github.com/zeromq/libzmq.git
WORKDIR /libzmq
# Build and install ZeroMQ using CMake
RUN mkdir build && cd build && \
	cmake .. && \
	make -j4 && \
	make install && \
	ldconfig

# Stage 2: Build IJavaScript kernel and native Node.js addons
# This stage sets up the JavaScript kernel for Jupyter
FROM node:20-bullseye AS ijavascript-builder
# Copy ZeroMQ libraries from the previous stage
COPY --from=zmq-builder /usr/local/ /usr/local/
# Install Python and build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
	python3 \
	python3-pip \
	build-essential \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*
ENV PATH="/root/.local/bin:${PATH}"
# Install JupyterLab and IJavaScript kernel
RUN python3 -m pip install --upgrade pip && \
	python3 -m pip install --no-cache-dir jupyterlab
# Install IJavaScript globally with external ZeroMQ support
RUN npm install -g ijavascript@latest --zmq-external
RUN npm install -g hnswlib-node
RUN ijsinstall --install=global

# Stage 3: Final Runtime Image
# This stage creates the minimal production image with all necessary components
FROM node:20-slim
WORKDIR /app
# Install minimal runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
	python3 \
	python3-pip \
	libzmq3-dev \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*
ENV PATH="/root/.local/bin:${PATH}"
# Configure Node.js to find globally installed packages
ENV NODE_PATH="/usr/local/lib/node_modules"

# Install JupyterLab in the final stage
RUN python3 -m pip install --upgrade pip --break-system-packages && \
	python3 -m pip install --no-cache-dir --break-system-packages jupyterlab

# Copy necessary libraries and components from previous stages
COPY --from=zmq-builder /usr/local/lib/libsodium* /usr/local/lib/
COPY --from=zmq-builder /usr/local/lib/libzmq* /usr/local/lib/
COPY --from=ijavascript-builder /usr/local/lib/node_modules/ /usr/local/lib/node_modules/
COPY --from=ijavascript-builder /usr/local/bin/ /usr/local/bin/
COPY --from=ijavascript-builder /usr/local/share/jupyter/kernels/javascript /usr/local/share/jupyter/kernels/javascript
# Update shared library cache
RUN ldconfig

# Install LangChain and its dependencies globally
# These packages are required for various AI/ML operations
RUN npm init -y
RUN npm install -g \
	langchain@latest \
	@langchain/core@latest \
	@langchain/community@latest \
	@langchain/openai@latest \
	@langchain/google-genai@latest \
	@langchain/anthropic@latest \
	@langchain/cohere@latest \
	@langchain/mistralai@latest \
	@langchain/groq@latest \
	@langchain/aws@latest \
	axios \
	cheerio \
	csv-parse \
	dotenv \
	pdf-parse \
	zod

# Expose JupyterLab port
EXPOSE 8888
# Start JupyterLab server
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root"]
