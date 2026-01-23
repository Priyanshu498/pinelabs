# pinelabs

# Jenkins Pipeline Migration Automation

A production-grade Python automation tool for migrating Jenkins pipeline jobs from Bitbucket to GitHub with comprehensive rollback support and detailed reporting.

## 🎯 Features

- **Recursive Folder Traversal**: Automatically discovers all pipeline jobs across Jenkins folders and subfolders
- **Intelligent URL Matching**: Supports multiple URL formats and partial matching
- **Automatic Rollback**: Backs up configurations and automatically rolls back on failure
- **Comprehensive Reporting**: Generates detailed reports in JSON, CSV, and console formats
- **Dry Run Mode**: Test migrations without making actual changes
- **Safe Execution**: Skips disabled jobs and validates configurations before updating
- **Idempotent**: Safe to re-run multiple times

## 📋 Prerequisites

- Python 3.8 or higher
- Jenkins API token with appropriate permissions
- Access to Jenkins instance
- Repository mapping configuration file

## 🚀 Installation

1. **Clone or download the project**:
```bash
git clone <repository-url>
cd jenkins-migration
```

2. **Install dependencies**:
```bash
pip install -r requirements.txt
```

3. **Create required directories**:
```bash
mkdir -p config backups logs reports
```

## ⚙️ Configuration

### Environment Variables

Set the following environment variables before running the script:

```bash
# Required
export JENKINS_URL="https://jenkins.example.com"
export JENKINS_USER="your-username"
export JENKINS_TOKEN="your-api-token"

# Optional
export REPO_MAPPING_FILE="config/repo_mapping.yaml"  # Default: config/repo_mapping.yaml
export DRY_RUN="true"  # Default: false
```

**Getting a Jenkins API Token:**
1. Log in to Jenkins
2. Click your username (top right)
3. Click "Configure"
4. Under "API Token", click "Add new Token"
5. Copy the generated token

### Repository Mapping File

Create a `config/repo_mapping.yaml` file with your Bitbucket to GitHub URL mappings:

```yaml
# Full SSH URL
"ssh://git@bitbucket.org/mycompany/backend-api.git": "git@github.com:mycompany/backend-api.git"

# HTTPS URL
"https://bitbucket.org/mycompany/frontend.git": "https://github.com/mycompany/frontend.git"

# Short format (partial match)
"bitbucket.org/mycompany/mobile-app": "git@github.com:mycompany/mobile-app.git"
```

**Supported formats:**
- SSH: `ssh://git@bitbucket.org/org/repo.git`
- HTTPS: `https://bitbucket.org/org/repo.git`
- Short: `bitbucket.org/org/repo`

The script intelligently matches URLs, so you can use the format most convenient for your use case.

## 🏃 Usage

### Basic Execution

```bash
python main.py
```

### Dry Run (Recommended First)

Test the migration without making changes:

```bash
export DRY_RUN="true"
python main.py
```

### Complete Example

```bash
#!/bin/bash

# Set environment variables
export JENKINS_URL="https://jenkins.example.com"
export JENKINS_USER="admin"
export JENKINS_TOKEN="11234567890abcdef1234567890abcdef"
export REPO_MAPPING_FILE="config/repo_mapping.yaml"

# Optional: Run in dry run mode first
export DRY_RUN="true"
python main.py

# If dry run looks good, run actual migration
export DRY_RUN="false"
python main.py
```

## 📊 Output

### Console Output

The script provides real-time progress updates:

```
2025-01-21 10:15:23 - INFO - Jenkins Pipeline Migration Automation Started
2025-01-21 10:15:23 - INFO - Loading repository mapping from: config/repo_mapping.yaml
2025-01-21 10:15:23 - INFO - Loaded 5 repository mappings
2025-01-21 10:15:24 - INFO - Successfully connected to Jenkins
2025-01-21 10:15:24 - INFO - Found 42 jobs to process
2025-01-21 10:15:25 - INFO - Processing job: folder1/pipeline-job-1
2025-01-21 10:15:26 - INFO - Successfully updated folder1/pipeline-job-1
...

================================================================================
MIGRATION SUMMARY
================================================================================
Total Jobs Scanned:      42
Successfully Updated:    38
Skipped:                 3
Failed:                  0
Rolled Back:             1
================================================================================
Success Rate:            90.48%
```

### Reports

Three types of reports are automatically generated:

1. **JSON Report**: `reports/migration_report_YYYYMMDD_HHMMSS.json`
   - Machine-readable format
   - Complete job details
   - Easy to parse for further processing

2. **CSV Report**: `reports/migration_report_YYYYMMDD_HHMMSS.csv`
   - Spreadsheet-compatible
   - Can be imported into Excel/Google Sheets
   - Good for stakeholder reporting

3. **Console Summary**: Human-readable summary with statistics and job details

### Logs

Detailed logs are saved to: `logs/migration_YYYYMMDD_HHMMSS.log`

### Backups

Original configurations are backed up to: `backups/YYYYMMDD_HHMMSS/`

Each backup includes:
- Original `config.xml` for each modified job
- `backup_manifest.json` with metadata

## 🔄 Rollback Behavior

### Automatic Rollback

If a job update fails, the script automatically:
1. Detects the failure
2. Retrieves the backed-up configuration
3. Restores the original configuration
4. Logs the rollback operation
5. Continues processing other jobs

### Manual Rollback

To manually rollback a specific job:

```python
from rollback import RollbackManager
from jenkins_client import JenkinsClient

# Initialize clients
jenkins_client = JenkinsClient(jenkins_url, username, token)
rollback_manager = RollbackManager('backups/20250121_101523')

# Rollback specific job
rollback_manager.rollback_job('folder1/my-job', jenkins_client)
```

### Batch Rollback

To rollback all jobs from a migration run, use the backup manifest:

```python
import json
from pathlib import Path

# Load backup manifest
backup_dir = Path('backups/20250121_101523')
with open(backup_dir / 'backup_manifest.json') as f:
    manifest = json.load(f)

# Rollback each job
for job_name in manifest['jobs']:
    rollback_manager.rollback_job(job_name, jenkins_client)
```

## 🏗️ Project Structure

```
jenkins-migration/
├── main.py                    # Main entry point
├── jenkins_client.py          # Jenkins API client
├── xml_updater.py             # XML configuration updater
├── rollback.py                # Rollback manager
├── report.py                  # Report generator
├── requirements.txt           # Python dependencies
├── README.md                  # This file
├── config/
│   └── repo_mapping.yaml      # Repository URL mappings
├── backups/                   # Backup storage (auto-created)
│   └── YYYYMMDD_HHMMSS/      # Timestamped backup directories
├── logs/                      # Log files (auto-created)
│   └── migration_YYYYMMDD_HHMMSS.log
└── reports/                   # Generated reports (auto-created)
    ├── migration_report_YYYYMMDD_HHMMSS.json
    └── migration_report_YYYYMMDD_HHMMSS.csv
```

## 🔍 How It Works

1. **Discovery Phase**:
   - Connects to Jenkins using API token
   - Recursively traverses all folders and subfolders
   - Identifies pipeline jobs (WorkflowJob, MultiBranchProject, etc.)
   - Skips disabled jobs and non-pipeline jobs

2. **Analysis Phase**:
   - Fetches `config.xml` for each job
   - Parses XML to locate SCM configuration
   - Checks if repository URL matches any Bitbucket patterns

3. **Backup Phase**:
   - Creates timestamped backup directory
   - Backs up original `config.xml` for each job to be modified
   - Generates backup manifest

4. **Update Phase**:
   - Replaces Bitbucket URLs with GitHub URLs
   - Validates updated XML
   - Pushes updated configuration to Jenkins

5. **Rollback Phase** (if needed):
   - Detects update failures
   - Retrieves backed-up configuration
   - Restores original configuration
   - Logs rollback status

6. **Reporting Phase**:
   - Collects statistics
   - Generates JSON, CSV, and console reports
   - Saves detailed logs

## 🛡️ Safety Features

- **Backups**: All original configurations are backed up before modification
- **Automatic Rollback**: Failed updates trigger automatic rollback
- **Dry Run Mode**: Test migrations without making changes
- **Validation**: XML validation before pushing to Jenkins
- **Skip Disabled Jobs**: Disabled jobs are automatically skipped
- **Idempotent**: Safe to run multiple times
- **Detailed Logging**: Complete audit trail of all operations
- **Error Handling**: Graceful error handling with descriptive messages

## ⚠️ Important Notes

1. **Test First**: Always run in dry run mode first
2. **Backup Jenkins**: Consider backing up entire Jenkins before large migrations
3. **Credentials**: Keep API tokens secure and never commit them to version control
4. **Permissions**: Ensure API token has permissions to read and update job configurations
5. **Network**: Ensure stable network connection to Jenkins
6. **Review Reports**: Always review generated reports after migration

## 🐛 Troubleshooting

### Connection Issues

**Problem**: `Failed to connect to Jenkins`

**Solutions**:
- Verify `JENKINS_URL` is correct and accessible
- Check that API token is valid
- Ensure network connectivity to Jenkins
- Verify Jenkins user has appropriate permissions

### URL Not Matching

**Problem**: Jobs are being skipped even though they use Bitbucket

**Solutions**:
- Check the exact URL format in Jenkins job config
- Add multiple URL format variations to mapping file
- Use short format (e.g., `bitbucket.org/org/repo`) for broader matching
- Enable debug logging to see what URLs are being compared

### Rollback Failures

**Problem**: Rollback fails after update failure

**Solutions**:
- Manually restore from backup directory
- Check Jenkins logs for specific errors
- Verify user has permissions to update job configuration
- Ensure backup files weren't corrupted or deleted

### Rate Limiting

**Problem**: Script slows down or fails with many jobs

**Solutions**:
- Add delays between API calls if needed
- Process jobs in smaller batches
- Check Jenkins server load

## 📝 Exit Codes

- `0`: Success (all jobs processed successfully)
- `1`: Failure (one or more jobs failed)

## 🤝 Contributing

Contributions are welcome! Please ensure:
- Code follows PEP 8 style guidelines
- All functions have docstrings
- Tests are included for new features
- Error handling is comprehensive

## 📄 License

This project is provided as-is for DevOps automation purposes.

## 🆘 Support

For issues or questions:
1. Check the troubleshooting section
2. Review log files in `logs/` directory
3. Examine backup files to verify configurations
4. Check Jenkins server logs for additional context

## 🔐 Security Considerations

- **API Tokens**: Store securely, rotate regularly
- **Network**: Use HTTPS for Jenkins connections
- **Logs**: Logs may contain sensitive information
- **Backups**: Protect backup directories appropriately
- **Credentials**: Never commit credentials to version controlMessage Priyasnhu yadav 
