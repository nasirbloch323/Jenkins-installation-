# Jenkins + Docker + Docker Compose Installation (Ubuntu)

## 1. System Update
```bash
sudo apt update -y
```

## 2. Install Docker
```bash
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```

Verify:
```bash
docker --version
docker compose version
```

## 3. Install Java (JDK 21 - required for Jenkins)
```bash
sudo apt install -y openjdk-21-jdk
java -version
```

## 4. Install Jenkins (WAR method - most reliable)
```bash
sudo mkdir -p /opt/jenkins
cd /opt/jenkins
sudo wget https://get.jenkins.io/war-stable/latest/jenkins.war

sudo useradd -m -d /var/lib/jenkins -s /bin/bash jenkins
sudo chown -R jenkins:jenkins /opt/jenkins
sudo usermod -aG docker jenkins
```

Create systemd service:
```bash
sudo tee /etc/systemd/system/jenkins.service > /dev/null << 'EOF'
[Unit]
Description=Jenkins Service
After=network.target

[Service]
Type=simple
User=jenkins
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64"
ExecStart=/usr/bin/java -jar /opt/jenkins/jenkins.war --httpPort=8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

Start Jenkins:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

## 5. Get Initial Admin Password
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## 6. Access Jenkins
```
http://<server-ip>:8080
```

## Notes
- Docker group membership requires re-login: `newgrp docker`
- Jenkins user must be in `docker` group to run docker-compose commands from pipelines
- Use `docker compose` (space) not `docker-compose` (hyphen) — new plugin syntax
