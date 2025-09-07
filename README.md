# RG28 Frappe Docker Development Environment

<div align="center">

<!-- ![RG28 Logo](https://via.placeholder.com/200x80/2563eb/ffffff?text=RG28) -->

**Enterprise-grade Frappe Framework development environment for RG28 company projects**

[![Docker](https://img.shields.io/badge/Docker-Required-blue?logo=docker)](https://www.docker.com/)
[![Frappe](https://img.shields.io/badge/Frappe-v15-green?logo=python)](https://frappeframework.com/)
[![License](https://img.shields.io/badge/License-Proprietary-red)](LICENSE)

</div>

---

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Development Setup](#development-setup)
- [RG28 Application Development](#rg28-application-development)
- [ERPNext Installation and Setup](#erpnext-installation-and-setup)
- [Development Workflow](#development-workflow)
- [Database Management](#database-management)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🎯 Overview

This repository contains a production-ready Docker development environment specifically designed for RG28 company's Frappe Framework projects. It provides a standardized, containerized development setup that ensures consistency across all RG28 development teams.

### ✨ Features

- **Containerized Environment**: Isolated development environment using Docker
- **Multi-Service Architecture**: MariaDB, Redis, Nginx, and Frappe Framework
- **RG28 Optimized**: Pre-configured for RG28-specific development workflows
- **Production Parity**: Development environment mirrors production setup
- **Easy Setup**: One-command environment initialization

## 🔧 Prerequisites

Before you begin, ensure you have the following installed on your system:

### Required Software

- **Docker Desktop** (v20.10+) - [Download](https://www.docker.com/products/docker-desktop)
- **Docker Compose** (v2.0+) - Usually included with Docker Desktop
- **Git** (v2.30+) - [Download](https://git-scm.com/downloads)

### System Requirements

- **RAM**: Minimum 8GB (16GB recommended)
- **Storage**: At least 10GB free space
- **OS**: macOS 10.15+, Windows 10+, or Linux (Ubuntu 20.04+)

### Verify Installation

```bash
# Check Docker installation
docker --version
docker compose version

# Check Git installation
git --version
```

## 🚀 Quick Start

### 1. Clone the Repository (Temporary)

```bash
git clone https://github.com/almariebu/rg28-frappe-docker.git
cd rg28-frappe-docker
```

### 2. Start the Development Environment

```bash
# Start all services in detached mode
docker compose -f .devcontainer-rg28/docker-compose.yml up -d

# Verify all containers are running
docker ps
```

### 3. Access the Development Container

```bash
# Connect to the Frappe development container
docker exec -e "TERM=xterm-256color" -w /workspace/development -it devcontainer-rg28-frappe-1 bash
```

## 🛠️ Development Setup

### Initialize Frappe Bench

Create a new Frappe bench instance for RG28 development:

```bash
# Initialize bench with Frappe v15 (latest stable commands)
bench init --skip-redis-config-generation --frappe-branch version-15 --python python3.11 frappe-bench-rg28

# Navigate to the bench directory
cd frappe-bench-rg28

# Set correct permissions (if needed)
sudo chown -R frappe:frappe /workspace/development/frappe-bench-rg28
```

### Configure Database and Redis Connections

Set up the containerized database and Redis configurations:

```bash
# Configure MariaDB connection for containerized environment
bench set-config -g db_host mariadb
bench set-config -g db_port 3306

# Configure Redis services for caching and queuing (v15 format)
bench set-config -g redis_cache "redis://redis-cache:6379"
bench set-config -g redis_queue "redis://redis-queue:6379"
bench set-config -g redis_socketio "redis://redis-queue:6379"

# Set additional Redis configurations for v15
bench set-config -g redis_cache_timeout 86400
bench set-config -g redis_queue_timeout 300

# Enable Redis usage
bench set-config -g use_redis_cache 1
bench set-config -g use_redis_queue 1

# Verify configurations
bench config show-config
```

### Configure Web Server Port

Update the Procfile to use the correct development port:

```bash
# Edit the Procfile
nano Procfile
```

**Change from:**
```
web: bench serve --port 8000
```

**To:**
```
web: bench serve --port 8011
```

### Create and Configure Development Site

```bash
# Create a new development site with admin password
bench new-site dev.local --admin-password admin123

# Set as default site for easier commands
bench use dev.local

# Alternative: Set as default site (legacy method)
bench set-default-site dev.local

# Enable developer mode for better debugging
bench --site dev.local set-config developer_mode 1

# Enable maintenance mode if needed
bench --site dev.local set-config maintenance_mode 0

# Verify site configuration
bench list-sites
```

## 📦 RG28 Application Development

### Option 1: Create New RG28 Application

If you're starting a new RG28 project:

```bash
# Create a new Frappe application for RG28
bench new-app rg28

# Install the application to your development site
bench --site dev.local install-app rg28

# Enable the app in hooks if needed
bench --site dev.local add-to-hosts
```

### Option 2: Use Existing RG28 Application

If you're working with an existing RG28 application:

```bash
# Clone existing RG28 application
bench get-app https://github.com/kabzko/rg28-app

# Install the application
bench --site dev.local install-app rg28

# Run any pending migrations
bench --site dev.local migrate
```

## 🏢 ERPNext Installation and Setup

### Adding ERPNext App to Your Bench

ERPNext is the flagship ERP application built on Frappe Framework. Here's how to add it to your development environment:

```bash
# Get ERPNext app (version 15 for Frappe v15)
bench get-app --branch version-15 erpnext

# Alternative: Get ERPNext from official repository
bench get-app https://github.com/frappe/erpnext --branch version-15

# Verify ERPNext app is added to bench
bench list-apps
```

### Installing ERPNext to Site

```bash
# Install ERPNext to your development site
bench --site dev.local install-app erpnext

# Run database migrations for ERPNext
bench --site dev.local migrate

# Clear cache after installation
bench --site dev.local clear-cache

# Restart bench to apply changes
bench restart
```

### ERPNext Initial Setup

After installation, you'll need to complete the ERPNext setup:

```bash
# Access ERPNext setup wizard
# Navigate to: http://localhost:8011
# Login with: Administrator / admin123

# Alternative: Run setup via command line
bench --site dev.local execute erpnext.setup.setup_wizard.setup_wizard.setup_complete \
    --args '{"language": "en", "country": "Philippines", "timezone": "Asia/Manila", 
    "currency": "PHP", "company_name": "RG28 Company", "company_abbr": "RG28"}'
```

### ERPNext Configuration for Development

```bash
# Enable developer mode for ERPNext customizations
bench --site dev.local set-config developer_mode 1

# Enable ERPNext specific developer features
bench --site dev.local set-config allow_tests 1

# Set up email configuration (optional for development)
bench --site dev.local set-config mail_server localhost
bench --site dev.local set-config mail_port 1025

# Enable print format builder
bench --site dev.local set-config enable_print_designer 1
```

### Adding Additional Frappe/ERPNext Apps

You can also add other popular Frappe apps alongside ERPNext:

```bash
# HR Management (HRMS)
bench get-app --branch version-15 hrms
bench --site dev.local install-app hrms

# Payments Integration
bench get-app --branch version-15 payments
bench --site dev.local install-app payments

# eCommerce Integration
bench get-app --branch version-15 ecommerce_integrations
bench --site dev.local install-app ecommerce_integrations

# Custom Reports and Dashboards
bench get-app --branch version-15 insights
bench --site dev.local install-app insights
```

### ERPNext Database Seeding (Optional)

For development purposes, you might want sample data:

```bash
# Install sample data for testing
bench --site dev.local execute erpnext.setup.demo.make_demo

# Import specific demo data
bench --site dev.local import-data demo

# Create sample customers, suppliers, items
bench --site dev.local execute erpnext.setup.demo.make_demo_data
```

### ERPNext Development Workflow

```bash
# Create custom DocTypes for ERPNext modules
bench --site dev.local make-doctype "RG28 Custom Customer" --module "Selling"

# Create custom fields in existing ERPNext DocTypes
bench --site dev.local make-custom-field "Customer" "rg28_customer_code" "Data"

# Create custom scripts for ERPNext forms
bench --site dev.local make-custom-script "Sales Order"

# Export ERPNext customizations
bench --site dev.local export-fixtures --app erpnext

# Backup ERPNext data
bench --site dev.local backup --with-files
```

### Application Structure

Your RG28 application should follow this structure:

```
rg28/
├── rg28/
│   ├── __init__.py
│   ├── config/
│   ├── modules/
│   ├── public/
│   └── templates/
├── hooks.py
├── modules.txt
└── setup.py
```

## 🔄 Development Workflow

### Starting Development

```bash
# Start the Frappe development server (v15 optimized)
bench start

# Alternative: Start with specific processes
bench start --skip-redis-config-generation

# Start with custom port
bench serve --port 8011

# Access your application at: http://localhost:8011
```

### Common Development Commands

#### Site Management (Updated for v15)

```bash
# Create new site with specific options
bench new-site mysite.local --db-name mysite_db --admin-password mypassword

# Set site configuration
bench --site dev.local set-config key value

# Get site configuration
bench --site dev.local get-config key

# Enable/disable maintenance mode
bench --site dev.local set-maintenance-mode on
bench --site dev.local set-maintenance-mode off

# Backup and restore with v15 improvements
bench --site dev.local backup --with-files --compress
bench --site dev.local restore backup_file.sql.gz --with-files
```

#### DocType Management

```bash
# Create a new DocType (v15 enhanced)
bench make-doctype "RG28 Custom DocType" --module "RG28 Core"

# Create DocType with custom fields and options
bench make-doctype "RG28 Customer" --fields "customer_name:Data:Required" "email:Email" "phone:Phone" --module "RG28 CRM"

# Generate DocType files with specific options
bench make-doctype "RG28 Product" --module "RG28 Products" --is-child-table 0 --is-tree 0

# Create child table DocType
bench make-doctype "RG28 Product Item" --module "RG28 Products" --is-child-table 1

# Export DocType for fixtures
bench --site dev.local export-doc "DocType" "RG28 Custom DocType"
```

#### Page and Report Creation

```bash
# Create a new Page with v15 enhancements
bench make-page "RG28 Dashboard" --module "RG28 Core"

# Create a new Report (Query Report)
bench make-report "RG28 Sales Report" --module "RG28 Reports" --type "Query Report"

# Create Script Report
bench make-report "RG28 Analytics" --module "RG28 Reports" --type "Script Report"

# Create a new Print Format
bench make-print-format "RG28 Invoice" --doctype "Sales Invoice" --module "RG28 Printing"

# Create Web Form
bench make-web-form "RG28 Lead Capture" --module "RG28 Web"

# Create Workspace
bench make-workspace "RG28 Operations" --module "RG28 Core"
```

#### Asset Management

```bash
# Build assets for production (v15 optimized)
bench build

# Build assets for specific app
bench build --app rg28

# Build assets for development with source maps
bench build --app rg28 --development

# Watch for asset changes during development
bench watch --app rg28

# Build and watch simultaneously
bench build --app rg28 --production --watch

# Clear build artifacts
bench clear-cache
bench clear-website-cache

# Compress assets for production
bench compress
```

#### Cache Management

```bash
# Clear all cache (v15 enhanced)
bench clear-cache

# Clear specific site cache
bench --site dev.local clear-cache

# Clear website cache only
bench --site dev.local clear-website-cache

# Clear specific cache types
bench --site dev.local clear-cache --doctype "Customer"

# Rebuild and clear
bench --site dev.local reload-doc all

# Clear Redis cache specifically
bench --site dev.local flush-redis

# Clear assets cache
bench clear-cache --build
```

### Code Quality and Testing

```bash
# Run tests for RG28 application
bench --site dev.local run-tests --app rg28

# Run specific test file
bench --site dev.local run-tests --app rg28 --module rg28.tests.test_custom

# Check code formatting
bench --site dev.local run-tests --app rg28 --coverage
```

## 🗄️ Database Management

### Backup Operations

```bash
# Create a full backup
bench backup

# Create backup with specific name
bench backup --with-files --compress

# Create backup for specific site
bench --site dev.local backup
```

### Restore Operations

```bash
# Restore from backup file
bench restore [backup-file-path]

# Restore with files
bench restore [backup-file-path] --with-files
```

### Importing Existing RG28 Database

If you have an existing RG28 database dump that you want to import into your Docker environment, follow this comprehensive guide:

#### Step 1: Prepare the Database File

```bash
# Verify the database file exists
ls -la /rg28_app/database/rg28-database.sql.gz

# Check file size to ensure it's not corrupted
file /rg28_app/database/rg28-database.sql.gz
```

#### Step 2: Copy Database File to Container

```bash
# Copy the database file to the Frappe container
docker cp /rg28_app/database/rg28-database.sql.gz \
    devcontainer-rg28-frappe-1:/workspace/development/rg28-database.sql.gz

# Verify the file was copied successfully
docker exec devcontainer-rg28-frappe-1 ls -la /workspace/development/rg28-database.sql.gz
```

#### Step 3: Import Database via Docker MariaDB

**Method A: Direct Import to MariaDB Container**

```bash
# Extract and import directly to MariaDB container
docker exec -i devcontainer-rg28-mariadb-1 sh -c \
    'gunzip < /dev/stdin | mysql -u root -padmin123' < \
    /rg28_app/database/rg28-database.sql.gz

# Alternative: Copy file to MariaDB container first
docker cp /rg28_app/database/rg28-database.sql.gz \
    devcontainer-rg28-mariadb-1:/tmp/rg28-database.sql.gz

# Then import from within MariaDB container
docker exec devcontainer-rg28-mariadb-1 sh -c \
    'gunzip -c /tmp/rg28-database.sql.gz | mysql -u root -padmin123'
```

**Method B: Import via Frappe Container with Bench**

```bash
# Connect to Frappe container
docker exec -it devcontainer-rg28-frappe-1 bash

# Navigate to development directory
cd /workspace/development/frappe-bench-rg28

# Create a new site for the imported database
bench new-site dev.local --admin-password admin123 --mariadb-root-password admin123

# Drop the newly created database (we'll replace it with our import)
bench --site dev.local drop-site --force

# Extract the database file
gunzip /workspace/development/rg28-database.sql.gz

# Import the database directly
mysql -h mariadb -u root -padmin123 < /workspace/development/rg28-database.sql

# Recreate site configuration pointing to imported database
bench new-site dev.local --admin-password admin123 --db-name [database_name_from_import]
```

#### Step 4: Configure Site for Imported Database

```bash
# If the database name is different, update site configuration
bench --site dev.local set-config db_name your_imported_db_name

# Update database host for containerized environment
bench --site dev.local set-config db_host mariadb
bench --site dev.local set-config db_port 3306

# Set Redis configurations
bench --site dev.local set-config redis_cache "redis://redis-cache:6379"
bench --site dev.local set-config redis_queue "redis://redis-queue:6379"
bench --site dev.local set-config redis_socketio "redis://redis-queue:6379"

# Enable developer mode
bench --site dev.local set-config developer_mode 1

# Set as default site
bench use dev.local
```

#### Step 5: Post-Import Configuration

```bash
# Run migrations to ensure database schema is up to date
bench --site dev.local migrate

# Clear all caches
bench --site dev.local clear-cache

# Rebuild website if needed
bench --site dev.local clear-website-cache

# Install any missing apps that were in the original database
bench --site dev.local list-apps
bench --site dev.local install-app [app_name] # if needed

# Verify the import was successful
bench --site dev.local console
```

#### Step 6: Verify Import Success

```bash
# Check if the site is accessible
bench --site dev.local browse

# Check database connection
bench --site dev.local mariadb

# Within MariaDB console, verify tables exist:
# SHOW DATABASES;
# USE your_database_name;
# SHOW TABLES;
# SELECT COUNT(*) FROM tabUser;

# Exit MariaDB console
# exit;

# Test site functionality
bench start
# Access: http://localhost:8011
```

#### Troubleshooting Database Import

```bash
# Check MariaDB container logs
docker logs devcontainer-rg28-mariadb-1

# Check if MariaDB is accepting connections
docker exec devcontainer-rg28-mariadb-1 mysql -u root -padmin123 -e "SHOW DATABASES;"

# Reset MariaDB if needed
docker restart devcontainer-rg28-mariadb-1

# Check disk space in containers
docker exec devcontainer-rg28-mariadb-1 df -h
docker exec devcontainer-rg28-frappe-1 df -h

# If import fails, check SQL file for errors
zcat /rg28_app/database/rg28-database.sql.gz | head -50
zcat /rg28_app/database/rg28-database.sql.gz | tail -50

# Check if file is corrupted
gunzip -t /rg28_app/database/rg28-database.sql.gz
```

#### Alternative: Using Bench Restore (Recommended)

If the SQL file was created using `bench backup`, use the bench restore method:

```bash
# Copy the backup file to the correct location
cp /rg28_app/database/rg28-database.sql.gz \
   /workspace/development/frappe-bench-rg28/sites/dev.local/private/backups/

# Restore using bench (this handles all configurations automatically)
bench --site dev.local restore \
   /workspace/development/frappe-bench-rg28/sites/dev.local/private/backups/rg28-database.sql.gz

# Run post-restore migrations
bench --site dev.local migrate
```

### Migration Operations

```bash
# Run database migrations (v15 syntax)
bench --site dev.local migrate

# Run migrations for specific app
bench --site dev.local migrate --app rg28

# Check migration status
bench --site dev.local migrate --dry-run

# Force migration (use with caution)
bench --site dev.local migrate --force

# Reset migrations (development only)
bench --site dev.local migrate --reset

# Show migration history
bench --site dev.local show-pending-migrations
```

### Database Utilities

```bash
# Export data
bench --site dev.local export-doc "RG28 Custom DocType" "DOC-001"

# Import data
bench --site dev.local import-doc [file-path]

# Reset database (⚠️ Use with caution)
bench --site dev.local reset
```

## 🐛 Troubleshooting

### Common Issues and Solutions

#### 1. Port Conflicts

**Problem**: Port 8011 is already in use

**Solution**:
```bash
# Check what's using the port
lsof -i :8011

# Kill the process or change port in Procfile
```

#### 2. Database Connection Issues

**Problem**: Cannot connect to MariaDB

**Solution**:
```bash
# Check if MariaDB container is running
docker ps | grep mariadb

# Restart MariaDB container
docker restart devcontainer-rg28-mariadb-1

# Check MariaDB logs
docker logs devcontainer-rg28-mariadb-1
```

#### 3. Redis Connection Issues

**Problem**: Redis services not accessible

**Solution**:
```bash
# Check Redis containers
docker ps | grep redis

# Restart Redis containers
docker restart devcontainer-rg28-redis-cache-1 devcontainer-rg28-redis-queue-1

# Verify Redis configuration
bench --site dev.local show-config | grep redis
```

#### 4. Permission Issues

**Problem**: File permission errors in mounted volumes

**Solution**:
```bash
# Fix permissions in the development directory
sudo chown -R $USER:$USER /workspace/development

# Or run container with proper user mapping
docker compose -f .devcontainer-rg28/docker-compose.yml down
docker compose -f .devcontainer-rg28/docker-compose.yml up -d
```

### Debug Commands

```bash
# Check overall system health
bench doctor

# Check site status
bench --site dev.local show-config

# View detailed logs
docker logs -f devcontainer-rg28-frappe-1

# Check container resource usage
docker stats
```

### Performance Optimization

```bash
# Enable Redis caching
bench set-config -g use_redis_cache 1

# Enable Redis session store
bench set-config -g session_expiry 86400

# Optimize database queries
bench set-config -g db_port 3306
```

## 🤝 Contributing

This development environment is maintained by the RG28 development team. We welcome contributions from RG28 team members.

### Contribution Guidelines

1. **Create an Issue**: Report bugs or suggest improvements
2. **Follow RG28 Standards**: Adhere to RG28 coding standards and conventions
3. **Test Thoroughly**: Ensure all changes work in the containerized environment
4. **Document Changes**: Update documentation for any new features
5. **Submit PR**: Create pull requests with detailed descriptions

### RG28 Development Standards

- **Code Style**: Follow PEP 8 for Python code
- **DocTypes**: Use RG28 naming conventions (e.g., `RG28 Customer`)
- **Modules**: Organize code in logical modules
- **Testing**: Write unit tests for new features
- **Documentation**: Include docstrings for all functions

### Getting Help

- **Internal Documentation**: Check RG28 internal wiki
- **Team Chat**: Use RG28 development Slack channel
- **Code Reviews**: Request reviews from senior developers
- **Mentorship**: Reach out to team leads for guidance

---

## 📞 Support

For technical support or questions about this development environment:

- **Email**: dev-support@rg28.com
- **Slack**: #rg28-dev-support
- **Internal Wiki**: [RG28 Development Hub](https://wiki.rg28.com/dev)

---

<div align="center">

**Built with ❤️ by the RG28 Development Team**

*This environment is proprietary to RG28 company and should not be shared externally.*

</div>

