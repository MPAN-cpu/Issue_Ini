# Google Sheets to GitHub Issues Automation

This GitHub Action automatically monitors a Google Sheet for changes in the "paper_id" column and creates new GitHub issues for each unique paper ID.

## Features

- 🔄 **Automated Monitoring**: Runs every 5 minutes via GitHub Actions
- 📊 **Google Sheets Integration**: Reads data from your Google Sheet
- 🎯 **Smart Issue Creation**: Creates GitHub issues with structured templates
- 🔍 **Duplicate Prevention**: Checks for existing issues before creating new ones
- 📝 **State Management**: Tracks processed paper IDs to avoid duplicates

## Setup Instructions

### 1. Google Sheets Setup

1. **Create a Google Service Account**:
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project or select existing one
   - Enable the Google Sheets API
   - Create a Service Account
   - Download the JSON key file

2. **Share Your Google Sheet**:
   - Open your Google Sheet
   - Share it with the service account email (found in the JSON key)
   - Give it "Viewer" permissions

3. **Prepare Your Sheet**:
   - Ensure your sheet has a "paper_id" column (default: Column A)
   - Add your paper IDs in this column
   - The first row can be a header (will be skipped)

### 2. GitHub Repository Setup

1. **Add Repository Secrets**:
   Go to your repository → Settings → Secrets and variables → Actions, and add:

   - `GOOGLE_SHEET_ID`: Your Google Sheet ID (from the URL)
   - `GOOGLE_SERVICE_ACCOUNT_KEY`: Base64 encoded service account JSON
   - `GITHUB_TOKEN`: Use the default `GITHUB_TOKEN` (automatically available)

2. **Encode Service Account Key**:
   ```bash
   # On macOS/Linux
   base64 -i your-service-account-key.json
   
   # Copy the output and paste it as GOOGLE_SERVICE_ACCOUNT_KEY secret
   ```

3. **Create Labels** (Optional):
   The action creates issues with labels `paper-review` and `automated`. You can create these labels in your repository:
   - Go to Issues → Labels → New label
   - Create: `paper-review` (suggested color: #0366d6)
   - Create: `automated` (suggested color: #d73a4a)

### 3. Customization

#### Adjust Sheet Column
If your "paper_id" column is not in column A, modify the `range_name` in `scripts/sheet_monitor.py`:

```python
# Change from 'A:A' to your column (e.g., 'B:B' for column B)
range_name = 'A:A'
```

#### Modify Issue Template
Edit the issue template in the `_create_github_issue` method in `scripts/sheet_monitor.py`.

#### Change Monitoring Frequency
Modify the cron schedule in `.github/workflows/sheet-monitor.yml`:

```yaml
schedule:
  # Current: every 5 minutes
  - cron: '*/5 * * * *'
  
  # Examples:
  # Every hour: '0 * * * *'
  # Every day at 9 AM: '0 9 * * *'
  # Every Monday: '0 9 * * 1'
```

## How It Works

1. **Scheduled Execution**: The GitHub Action runs every 5 minutes
2. **Sheet Reading**: Fetches data from your Google Sheet using the Google Sheets API
3. **State Check**: Compares current paper IDs with previously processed ones
4. **Duplicate Prevention**: Checks if an issue already exists for each paper ID
5. **Issue Creation**: Creates new GitHub issues with structured templates
6. **State Update**: Saves the list of processed paper IDs

## Issue Template

Each created issue includes:
- **Title**: "Paper Review: {paper_id}"
- **Labels**: `paper-review`, `automated`
- **Body**: Structured template with tasks and metadata

## Troubleshooting

### Common Issues

1. **"Error initializing Google Sheets API"**:
   - Check that `GOOGLE_SERVICE_ACCOUNT_KEY` is properly base64 encoded
   - Verify the service account has access to the sheet

2. **"No data found in sheet"**:
   - Ensure the sheet is shared with the service account email
   - Check that the column range is correct

3. **"Failed to create issue"**:
   - Verify `GITHUB_TOKEN` has issue creation permissions
   - Check repository permissions

### Manual Testing

You can manually trigger the workflow:
1. Go to Actions tab in your repository
2. Select "Monitor Google Sheet and Create Issues"
3. Click "Run workflow"

### Logs

Check the workflow logs in the Actions tab to see detailed execution information and any error messages.

## Security Notes

- The service account key is stored as a base64-encoded secret
- Only read permissions are required for the Google Sheet
- The action only creates issues, it doesn't modify your sheet

## Contributing

Feel free to submit issues and enhancement requests! 