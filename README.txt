# Install Docker - check official installation instructions for your system
# https://docs.docker.com/installation/

# Tutorial based on Ubuntu 14.04

# To be sure that You don't hit a docker bug with IPv6 port binding only - enable as root forwarding in sysctl.conf:
$ echo "net.ipv6.conf.all.forwarding=1" >> /etc/sysctl.conf
$ sysctl -p

# Provide/Install PostgreSQL and Redis server - example for Ubuntu 14.04 
$ apt-get install -y postgresql redis-server
$ echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/9.3/main/pg_hba.conf
$ echo "listen_addresses='*'" >> /etc/postgresql/9.3/main/postgresql.conf
$ service postgresql restart
$ sudo su - postgres
$ createuser --no-superuser --no-createdb --no-createrole zato2
$ createdb --owner=zato2 zato2
$ psql --dbname zato2 --command="ALTER ROLE zato2 WITH PASSWORD 'zato2'"
# Later in configs specify odb_host with IP address - no localhost
# Redis reconfiguration to listen on all interfaces
$ sed 's/bind 127./#bind 127./g' /etc/redis/redis.conf |sudo tee  /etc/redis/redis.conf >/dev/null
$ service redis-server restart

# Create Zato ODB and Cluster with Postgres as described in Docs:
https://zato.io/docs/2.0/admin/cli/create-odb.html
https://zato.io/docs/2.0/admin/cli/create-cluster.html
# or run this Dockerfile with configs which will create odb and cluster for You:
$ mkdir -p ~/zato-docker-odb-dev && cd ~/zato-docker-odb-dev && wget https://zato.io/download/docker/odb/Dockerfile \
    && wget https://zato.io/download/docker/odb_cluster/zato_odb.config \
    && wget https://zato.io/download/docker/odb_cluster/zato_cluster.config

# Example of zato_odb.config



# Example of zato_cluster.config


# change configs parameters and run:
$ sudo docker build --no-cache -t zato-odb-2.0.2 .

# Create certificates which will be needed for Zato components. If You like - You can use this script, which will create all selfsigned certs what You need:
$ mkdir -p ~/zato-cert-dev && cd ~/zato-cert-dev && wget https://zato.io/download/docker/gencert.sh && chmod +x gencert.sh && ./gencert.sh

# Copy propper certificates to destination catalogoues.

# Now, You can build components what You need:

# Build Zato Docker - Load Balancer image:
$ cd ~/zato-docker-lb-dev && wget https://zato.io/download/docker/load_balancer/Dockerfile \
 && wget https://zato.io/download/docker/load_balancer/zato_load_balancer.config

$ sudo docker build --no-cache -t zato-lb-2.0.2 .

# Create a container in which Zato Load balancer will be launched:
$ sudo docker run -it -p 11223:11223 -p 20150:20150 zato-lb-2.0.2

# Build Zato Docker - Web Admin image:
$ cd ~/zato-docker-wa-dev && wget https://zato.io/download/docker/web_admin/Dockerfile \
 && wget https://zato.io/download/docker/web_admin/zato_web_admin.config \
 && wget https://zato.io/download/docker/web_admin/zato_update_password.config

$ sudo docker build --no-cache -t zato-wa-2.0.2 .

# Create a container in which Zato Web Admin will be launched:
$ sudo docker run -it -p 8183:8183 zato-wa-2.0.2

# To check web admin password run:
$ sudo docker run -i -t zato-wa-2.0.2 cat web_admin_password

# Build Zato Docker - Server image:
$ cd ~/zato-docker-srv-dev && wget https://zato.io/download/docker/server/Dockerfile \
 && wget https://zato.io/download/docker/server/zato_web_admin.config

$ sudo docker build --no-cache -t zato-srv-2.0.2 .

# Create a container in which Zato Server will be launched:
$ sudo docker run -it -p 17010:17010 zato-srv-2.0.2

# After successfully create and run all containers with components You can run admin console in web browser i.e.:
http://localhost:8183
