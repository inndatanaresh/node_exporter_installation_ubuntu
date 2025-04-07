# node exporter for ubuntu 

# create a file named with "install_node_exporter.sh"
# paste the below script in the created file : 

-----------------------------------------------------------------------------------------------------------------------
#!/bin/bash

set -e

echo "===== Step 1: Create node_exporter user ====="
useradd -rs /bin/false node_exporter || echo "User already exists."

echo "===== Step 2: Download node_exporter v1.7.0 ====="
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz -P /tmp/

echo "===== Step 3: Extract the tar.gz file ====="
tar -xzf /tmp/node_exporter-1.7.0.linux-amd64.tar.gz -C /tmp

echo "===== Step 4: Move node_exporter binary to /usr/local/bin ====="
mv /tmp/node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
chown node_exporter:node_exporter /usr/local/bin/node_exporter
chmod +x /usr/local/bin/node_exporter

echo "===== Step 5: Allow port 9100 through UFW (Ubuntu Firewall) ====="
if command -v ufw &> /dev/null; then
    ufw allow 9100/tcp || true
else
    echo "ufw not found, skipping firewall config. If using another firewall, open port 9100 manually."
fi

echo "===== Step 6: Create systemd service file ====="
cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

echo "===== Step 7: Reload systemd daemon ====="
systemctl daemon-reload

echo "===== Step 8: Enable and start node_exporter service ====="
systemctl enable --now node_exporter

echo "===== Step 9: Restore SELinux context (optional for Ubuntu) ====="
if command -v restorecon &> /dev/null; then
    /sbin/restorecon -v /usr/local/bin/node_exporter
else
    echo "restorecon not found (SELinux likely not used), skipping SELinux step."
fi

echo "===== Step 10: Show node_exporter service status ====="
systemctl status node_exporter --no-pager

echo "✅ Node Exporter is installed and running on port 9100"
echo "🌐 Visit http://<your_server_ip>:9100/metrics to verify."

--------------------------------------------------------------------------------------------------------------

# run the below commands with sudo 

  chmod +x install_node_exporter.sh

  sudo ./install_node_exporter.sh

  Visit http://<your_server_ip>:9100/metrics to verify."
