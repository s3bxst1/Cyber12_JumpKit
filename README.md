# Cyber12_JumpKit
This is a jump kit used for incedent response in small businesses. In case of a complete network blackout, companies can use this to get a containerized network setup off of one laptop/computer



# Jump Kit Incident Response Deployment Guide

## Overview

This jump kit provides a comprehensive Docker-based incident response infrastructure designed for small businesses with approximately 100 employees. The system creates an isolated network environment that can be rapidly deployed during cybersecurity incidents.

## Prerequisites

### System Requirements
- Operating System: Linux (Ubuntu 20.04+ recommended), macOS, or Windows with WSL2
- RAM: Minimum 8GB (16GB recommended for 100 users)
- Storage: At least 20GB free space
- Network: Internet connection for initial setup

### Required Software
1. Docker (version 20.10+)
2. Docker Compose (version 2.0+)
3. Git (for version control)

## Installation

### Step 1: Install Docker and Docker Compose

#### On Ubuntu/Debian:
```bash
# Update system packages
sudo apt update

# Install Docker
sudo apt install -y docker.io docker-compose

# Add your user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or run:
newgrp docker

# Verify installation
docker --version
docker-compose --version
```

#### On macOS:
```bash
# Install Docker Desktop from https://www.docker.com/products/docker-desktop
# Or use Homebrew:
brew install --cask docker
brew install docker-compose
```

#### On Windows:
1. Install Docker Desktop from https://www.docker.com/products/docker-desktop
2. Enable WSL2 integration
3. Open PowerShell or Command Prompt as Administrator

### Step 2: Create Project Directory

```bash
# Create a dedicated directory for the jump kit
mkdir ~/jumpkit-incident-response
cd ~/jumpkit-incident-response

# Verify you're in the right directory
pwd
```

### Step 3: Create Configuration Files

Create the main Docker Compose configuration file:

```bash
nano docker-compose.yml
```

Copy the Docker Compose configuration from the provided template.

Create the setup script:

```bash
nano setup.sh
```

Copy the setup script content from the provided template.

Make the setup script executable:

```bash
chmod +x setup.sh
```

### Step 4: Initialize the Environment

Run the setup script to create all necessary directories and configuration files:

```bash
./setup.sh
```

### Step 5: Deploy the Jump Kit

Start all services:

```bash
./start_jumpkit.sh
```

## Service Access

Once deployed, the following services will be available:

- Main Dashboard: http://localhost:8000
- Chat Server (Mattermost): http://localhost:8082
- File Server: http://localhost:8081
- Log Server (Graylog): http://localhost:9000
- DNS Admin (Pi-hole): http://localhost:8080

## Default Credentials

- File Server: responder / responder
- DNS Admin: admin / jumpkit2024!
- Log Server: admin / admin
- Database: responder / secure_response_2024

**IMPORTANT: Change all default passwords immediately after deployment**

## Verification

### Check Container Status
```bash
# Check that all containers are running
docker-compose ps

# View logs for specific service
docker-compose logs [service_name]

# View all logs
docker-compose logs -f
```

### Test Database Connection
```bash
# Connect to database
docker exec -it jumpkit_db psql -U responder -d incident_response

# Check tables
\dt

# Exit database
\q
```

### Test Backup System
```bash
# Run manual backup
./scripts/backup.sh

# Check backup was created
ls -la backups/
```

## Configuration

### Chat Server (Mattermost)
1. Navigate to http://localhost:8082
2. Create admin account when prompted
3. Set up your incident response team
4. Create channels: #general, #incidents, #evidence, #communications

### File Server
1. Navigate to http://localhost:8081
2. Login with default credentials
3. Create folders: evidence, reports, documentation, backups
4. Set appropriate permissions

### DNS Server (Pi-hole)
1. Navigate to http://localhost:8080
2. Login with default credentials
3. Configure DNS filtering rules as needed
4. Set up custom DNS entries for internal services

### Log Server (Graylog)
1. Navigate to http://localhost:9000
2. Login with default credentials
3. Set up log inputs for your environment
4. Configure dashboards for incident monitoring

## Customization

### Update Emergency Contacts
```bash
# Connect to database
docker exec -it jumpkit_db psql -U responder -d incident_response

# Update contacts table with your information
UPDATE contacts SET 
    email = 'your-it@company.com', 
    phone = '+1-555-YOUR-NUMBER' 
WHERE role = 'System Administrator';

# Exit database
\q
```

### Customize Web Dashboard
```bash
# Edit the main dashboard
nano web/index.html

# Update company information, contacts, and branding
```

### Add Custom Documentation
```bash
# Create your incident response procedures
nano web/docs/playbook.html

# Add your specific procedures and contact information
```

## Maintenance

### Daily Operations
```bash
# Check system status
docker-compose ps

# View logs
docker-compose logs -f

# Backup data
./scripts/backup.sh
```

### Updates
```bash
# Pull latest images
docker-compose pull

# Restart with new images
docker-compose down
docker-compose up -d
```

### Cleanup
```bash
# Stop all services
docker-compose down

# Remove all data (CAUTION - THIS IS DESTRUCTIVE)
docker-compose down -v
sudo rm -rf postgres_data/ pihole/ mattermost/ graylog/ backups/
```

## Troubleshooting

### Common Issues

#### Port Conflicts
```bash
# Check what's using a port
sudo netstat -tlnp | grep :8000

# Stop conflicting services or change ports in docker-compose.yml
```

#### Permission Issues
```bash
# Fix file permissions
sudo chown -R $USER:$USER ./
chmod -R 755 ./
```

#### Container Won't Start
```bash
# Check logs for specific container
docker-compose logs [container_name]

# Restart specific service
docker-compose restart [service_name]
```

#### Database Connection Issues
```bash
# Reset database
docker-compose down
sudo rm -rf postgres_data/
docker-compose up -d database
```

### Emergency Shutdown
```bash
# Graceful shutdown
docker-compose down

# Emergency stop (kills containers)
docker-compose kill
```

## Security Considerations

### Network Security
- Change all default passwords immediately
- Use strong passwords for all services
- Consider running behind a VPN for remote access
- Enable HTTPS certificates for production use

### Data Protection
- Encrypt backup storage
- Implement proper access controls
- Regular security updates
- Monitor access logs

### Incident Response Best Practices
- Test the system regularly with tabletop exercises
- Keep emergency contact information updated
- Document all customizations
- Train team members on system usage

## Architecture

The jump kit consists of the following components:

### Core Services
- **DNS Server (Pi-hole)**: Network traffic control and monitoring
- **Web Server (Nginx)**: Central dashboard and documentation hub
- **File Server**: Secure evidence and document sharing
- **Database (PostgreSQL)**: Incident tracking and evidence logging
- **Chat Server (Mattermost)**: Secure team communication
- **Log Server (Graylog)**: Centralized log analysis
- **Network Monitor**: Traffic analysis tools
- **Security Scanner (OWASP ZAP)**: Vulnerability assessment
- **Backup Service**: Data protection and recovery

### Network Configuration
- All services run on isolated Docker network (172.20.0.0/16)
- Static IP addresses assigned to each service
- Internal DNS resolution between services
- External access via mapped ports

### Data Storage
- Evidence files stored in shared volumes
- Database for incident tracking and timeline
- Automated backup system with versioning
- Log aggregation and retention

## File Structure

```
jumpkit-incident-response/
├── docker-compose.yml          # Main configuration
├── setup.sh                    # Environment setup script
├── start_jumpkit.sh            # Service startup script
├── nginx.conf                  # Web server configuration
├── init.sql                    # Database initialization
├── filebrowser.json            # File server configuration
├── web/                        # Web dashboard files
│   ├── index.html
│   └── docs/                   # Documentation
├── files/                      # Shared file storage
├── scripts/                    # Utility scripts
│   └── backup.sh
├── backups/                    # Backup storage
├── logs/                       # System logs
├── postgres_data/              # Database storage
├── pihole/                     # DNS server data
├── mattermost/                 # Chat server data
├── graylog/                    # Log server data
└── scan_results/               # Security scan results
```

## Support

For technical issues:
1. Check the troubleshooting section above
2. Review container logs using docker-compose logs
3. Verify system requirements are met
4. Contact your IT team or system administrator

For incident response procedures:
1. Reference the documentation in web/docs/
2. Follow your organization's incident response plan
3. Contact emergency contacts listed in the dashboard

## License

This jump kit is provided as-is for incident response purposes. Ensure compliance with your organization's policies and applicable laws when deploying and using this system.
