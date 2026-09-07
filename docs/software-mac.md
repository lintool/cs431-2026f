# CS 431 (Fall 2026): Software Setup for Mac

This page provides instructions on setting up the compute environment necessary for CS 431 (Fall 2026) for the Mac.

## Preliminaries

We will be using Conda to manage your environment.
Make sure you have Conda installed already.

We will be using Python 3.12 and Java 21.
Please make sure you use these versions.

## Spark

Go to [the Spark download page](https://spark.apache.org/downloads.html) and download Spark 4.2.0 (Pre-built for Apache Hadoop 3.4 and later).
Unpack the tarball somewhere on your machine and set the following environment variables.

```bash
export SPARK_HOME="/path/to/spark-4.2.0-bin-hadoop3"
export PATH="$PATH:$SPARK_HOME/bin"
```

Obviously, you'll want to change `SPARK_HOME` to the actual location of the unpacked directory.

## Conda Environment

Use Conda to create the `cs431` environment:

```bash
conda deactivate
conda env list
conda create -n cs431 python=3.12 -y
conda activate cs431

conda install -c conda-forge openjdk=21 maven -y
conda install -c conda-forge jupyterlab -y
conda install -c conda-forge postgresql -y

pip install pyspark findspark psutil
pip install sqlalchemy psycopg
pip install pandas numpy
```

Note that you only do the above **once**.
Subsequently, use the following to activate your `cs431` environment:

```bash
conda activate cs431
```

If you want to remove your environment (for whatever reason) and start over:

```bash
conda deactivate
conda env list
conda env remove --name cs431
```

## Spark Smoke Test

To make sure the JVM is installed:

```bash
java --version
```

You should see JDK 21 version info.

To make sure PySpark works:

```bash
pyspark --version
```

You should get a Spark banner and some version information.
You should not see any errors (warnings okay).
This confirms that PySpark is set up and configured properly.

## PostgreSQL

You'll also need PostgreSQL (`psql`) for this course.
Check installation:

```bash
% psql --version
psql (PostgreSQL) 18.6
```

In a fresh install, the PostgreSQL server should _not_ be ready:

```bash
% pg_isready -h 127.0.0.1 -p 5432
127.0.0.1:5432 - no response
```

Activate conda and initialize a new PostgreSQL data directory **once**:

```bash
# Make sure you're in the cs431 conda environment
conda activate cs431
initdb -D "$HOME/pgdata-cs431" --encoding=UTF8
```

Then start the server:

```bash
pg_ctl -D "$HOME/pgdata-cs431" \
  -l "$HOME/pgdata-cs431/server.log" \
  -o "-h 127.0.0.1 -p 5432" start
```

At this point, the PostgreSQL server should be ready:

```bash
% pg_isready -h 127.0.0.1 -p 5432
127.0.0.1:5432 - accepting connections
```

You need to create the `cs431` user, which is required by the assignment.
You only need to do this once.

```bash
psql -h 127.0.0.1 -d postgres -c "CREATE ROLE cs431 WITH LOGIN CREATEDB;"
```

Verify that the `cs431` user exists:

```bash
psql -h 127.0.0.1 -d postgres -c "SELECT rolname, rolcanlogin, rolcreatedb FROM pg_roles WHERE rolname = 'cs431';"
```

Sanity checks:

```bash
psql -h 127.0.0.1 -p 5432 -d postgres -c 'SELECT version();'
psql -h 127.0.0.1 -p 5432 -d postgres -c 'SELECT current_user';
```

At this point, `psql` should be set up and configured properly.

To stop the server:

```bash
pg_ctl -D "$HOME/pgdata-cs431" stop
```

The next time you need to access `psql`, you'll need to restart the server.

## Jupyter Notebook

Make sure you're in the `cs431` Conda environment.
Then, start Jupyter Notebook:

```bash
jupyter lab
```

You should now be able to access Jupyter Notebook in your browser.
