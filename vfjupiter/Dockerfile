# Debian 11 is recommended
FROM debian:11-slim

# Suppress interactive prompts
ENV DEBIAN_FRONTEND=noninteractive

# (Required) Install utilities required by Spark scripts
RUN apt update && apt install -y procps tini libjemalloc2 wget gnupg

# Enable jemalloc2 as default memory allocator
ENV LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2

# (Optional) Add extra jars
ENV SPARK_EXTRA_JARS_DIR=/opt/spark/jars/
ENV SPARK_EXTRA_CLASSPATH='/opt/spark/jars/*'
RUN mkdir -p "${SPARK_EXTRA_JARS_DIR}"
# TODO Put JAR names in variables
RUN wget https://repo1.maven.org/maven2/io/delta/delta-core_2.13/2.4.0/delta-core_2.13-2.4.0.jar -P "${SPARK_EXTRA_JARS_DIR}" \
  && wget https://repo1.maven.org/maven2/io/delta/delta-storage/2.4.0/delta-storage-2.4.0.jar -P "${SPARK_EXTRA_JARS_DIR}"

# Install and configure Miniconda3
ENV CONDA_HOME=/opt/miniconda3
ENV PYSPARK_PYTHON=${CONDA_HOME}/bin/python
ENV PATH=${CONDA_HOME}/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# TODO Pin version of Miniconda
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
  && bash Miniconda3-latest-Linux-x86_64.sh -b -p /opt/miniconda3 \
  && conda config --system --set always_yes True \
  && conda config --system --set auto_update_conda False \
  && conda config --system --prepend channels conda-forge \
  && conda config --system --set channel_priority strict \
  && conda config --set solver libmamba

# Install required Conda packages for Dataproc Serverless
# TODO Pin package versions
RUN conda install -n base -c conda-forge \
  cython \
  fastavro \
  fastparquet \
  gcsfs \
  google-cloud-bigquery-storage \
  google-cloud-bigquery[pandas] \
  google-cloud-container \
  google-cloud-dataproc \
  google-cloud-logging \
  google-cloud-monitoring \
  google-cloud-pubsub \
  google-cloud-storage \
  koalas \
  matplotlib \
  nltk \
  numba \
  numpy \
  openblas \
  orc \
  pandas \
  pip \
  pyarrow \
  pysal \
  pytables \
  python \
  regex \
  requests \
  rtree \
  seaborn \
  sqlalchemy \
  sympy \
  virtualenv \
  && conda clean -afy

# Install extra Conda packages
RUN conda install -n base -c conda-forge \
  PyYAML \
  lxml \
  schema \
  validators \
  hvac \
  python-gnupg \
  delta-spark \
  && conda clean -afy

# Set up authentication to Google Artifact Registry, set up keyring and install extra Pip packages
# TODO Pin package versions
# ARG GCP_PROJECT_ID
# RUN --mount=type=secret,id=gcp_gar_key,target=/gcp_gar_key.json \
#   export GOOGLE_APPLICATION_CREDENTIALS=/gcp_gar_key.json \
#   && pip install keyring keyrings.google-artifactregistry-auth \
#   && pip install --index-url https://europe-west2-python.pkg.dev/${GCP_PROJECT_ID}/${GCP_PROJECT_ID}-python-repository/simple \
#     jupiter-common-utils

# (Required) Create the 'spark' group/user
# The GID and UID must be 1099. Home directory is required
RUN groupadd -g 1099 spark
RUN useradd -u 1099 -g 1099 -d /home/spark -m spark

# Add application code
# ARG PIPELINE_DIR
# COPY ${PIPELINE_DIR} /src

USER spark
