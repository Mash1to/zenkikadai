1. ssh ec2-user@54.163.34.250 -i C:\Users\ktc\Downloads\suiyou12gen.pem
2. sudo yum install vim -y
3. sudo yum install -y docker
4. sudo systemctl start docker
5. sudo systemctl enable docker
6. sudo usermod -a -G docker ec2-user
7. sudo mkdir -p /usr/local/lib/docker/cli-plugins/
8. sudo curl -SL https://github.com/docker/compose/releases/download/v2.36.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
9. sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

