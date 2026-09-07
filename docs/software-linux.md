# CS 431 (Fall 2026): Software Setup for Linux

This page provides instructions on setting up the compute environment for CS 431 (Fall 2026) in the Student Linux Environment.

## Preliminaries

Make sure you can ssh into `linux.student.cs.uwaterloo.ca`.
All the instructions below require you to be logged into the Student Linux Environment.

Note that the Student Linux environment actually represents a collection of Linux servers; see [this page for the actual physical hosts](https://cs.uwaterloo.ca/cscf/teaching-hosts).
This means that two ssh instances might not be logging you into the same physical machine.
To find out which physical host you are on:

```bash
$ hostname -f
ubuntu2404-012.student.cs.uwaterloo.ca
```

For example, the above command tells me I'm on `ubuntu2404-012.student.cs.uwaterloo.ca`.
To make sure you're on the same physical machine (e.g., to talk to the PostgreSQL server), you can ssh directly into a physical host.

We will be using Conda to manage your environment.
Make sure you have Conda installed already.

We will be using Python 3.12 and Java 21.
Please make sure you use these versions.

## Spark

Go to [the Spark download page](https://spark.apache.org/downloads.html) and download Spark 4.2.0 (Pre-built for Apache Hadoop 3.4 and later).
Unpack the tarball.

Something like the following works, but adjust to suit your needs accordingly:

```bash
wget https://archive.apache.org/dist/spark/spark-4.2.0/spark-4.2.0-bin-hadoop3.tgz
tar -xvzf spark-4.2.0-bin-hadoop3.tgz -C $HOME
```

Set the following environment variables.

```bash
export SPARK_HOME="/path/to/spark-4.2.0-bin-hadoop3"
export PATH="$PATH:$SPARK_HOME/bin"
```

Obviously, you'll want to change `SPARK_HOME` to the actual location of the unpacked directory.

## Conda Environment

Miniconda is not available by default in the Student Linux Environment.
Try `which conda`: if `conda` is already installed, then you can skip this step.

Otherwise, you'll need to install Conda:

```bash
# Download Miniconda installer
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# Run the installer
bash ./Miniconda3-latest-Linux-x86_64.sh
# Reload shell
source ~/.bashrc
```

The following only needs to be done **once**.

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
psql (PostgreSQL) 18.4
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

(If you are still getting errors, particularly from another ssh instance, make sure you're on the same physical machine: see detailed instructions above.)

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

This starts Jupyter Notebook on the (remote) machine, but since you're in the Student Linux environment, there are extra steps to gain access from the browser on your local machine.
There are many ways to accomplish this, but the recommended way is to start an ssh tunnel.

Once you start Jupyter Notebook, the shell is occupied, so you have to do the following in another shell.
Make sure you've ssh'ed into the same physical machine (see above).

We need to find the source IP used for outbound traffic:

```bash
$ ip route get 1.1.1.1
1.1.1.1 via 129.97.167.129 dev enp1s0 src 129.97.167.157 uid 2537
    cache
```

Look for the address after `src`, that's the machine's IP.
In this case, the IP address is `129.97.167.157`.

Create a tunnel from port 8888 on the remote machine (running Jupyter Notebook) to local port 8811:

```bash
ssh -N -L 8811:localhost:8888 jimmylin@129.97.167.157
```

You can change the local port 8811 to suit your needs; use the IP from above.
Obviously, change your username.

After you set up the tunnel, you should be able to navigate to `http://localhost:8811/` in the browser on your (local) machine to access Jupyter Notebook.
When you log in the first time, it'll ask for a token.
Check the shell running Jupyter Notebook for the token.
