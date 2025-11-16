# WinGet Application Import/Export Manager

A PowerShell GUI application for backing up and restoring your Windows applications using Microsoft's WinGet package manager.

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![PowerShell](https://img.shields.io/badge/PowerShell-5.0+-green.svg)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg)
![License](https://img.shields.io/badge/license-MIT-orange.svg)

---

## 📋 Executive Summary

**WinGet Application Manager** provides a user-friendly graphical interface to export your currently installed Windows applications to a portable JSON format and selectively restore them on the same or different computer. Perfect for:

- 🔄 **System migrations** - Moving to a new computer
- 💾 **Backup and restore** - Creating application snapshots before major system changes
- 🏢 **IT deployments** - Standardising application installations across multiple machines
- 🎯 **Selective installation** - Choose exactly which applications to install from your backup

### Key Features

✅ **One-click export** of all installed applications  
✅ **Interactive grid view** to review and select applications before installation  
✅ **Dual output formats** - JSON (for import) and CSV (for documentation)  
✅ **UK date formatting** throughout the interface  
✅ **Progress tracking** with real-time installation feedback  
✅ **Source filtering** - Automatically identifies which apps can be reinstalled via WinGet  
✅ **Batch operations** - Select/deselect all applications with one click

---

## 🚀 Quick Start

### Prerequisites

- **Windows 10/11** (version 1809 or later)
- **PowerShell 5.0** or later (included with Windows)
- **WinGet** package manager ([installation instructions](https://learn.microsoft.com/en-us/windows/package-manager/winget/))

### Installation

1. Download `WinGetAppManager.ps1` from this repository
2. Right-click the file and select **"Run with PowerShell"**
   - If you encounter execution policy errors, run: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

### Basic Usage

#### Export Applications
1. Click **"Export Applications"**
2. Choose a save location (default: `C:\temp\MyApps_DD-MM-YY.json`)
3. Wait for the export to complete
4. Review the generated files:
   - **JSON file** - Used for importing applications
   - **CSV file** - Human-readable list with exportable status

#### Import Applications
1. Click **"Load Import File"**
2. Select your JSON export file
3. Review the applications in the grid view
4. Check/uncheck applications you want to install
5. Click **"Install Selected"**
6. Monitor the installation progress in the verbose output window

---

## 📖 Technical Documentation

### Architecture

The application is built using:
- **WPF (Windows Presentation Foundation)** for the GUI
- **PowerShell scripting** for WinGet integration
- **JSON Schema 2.0** for WinGet compatibility

### How It Works

#### Export Process

1. **Discovery Phase**
   - Executes `winget list` command
   - Captures all installed applications from all sources (winget, msstore, etc.)

2. **Parsing Phase**
   - Uses regex pattern matching to extract application metadata:
     ```regex
     ^(?<Name>.*?)\s{2,}(?<Id>\S+)\s{2,}(?<Version>\S*)\s{2,}(?<Source>\S+)$
     ```
   - Creates structured objects with: Name, ID, Version, Source

3. **Filtering Phase**
   - Identifies applications from WinGet source (can be reinstalled)
   - Flags applications from other sources as non-exportable

4. **Output Generation**
   - **JSON Manifest**: WinGet-compatible format with schema validation
   - **CSV Report**: Complete application list with exportability status

#### Import Process

1. **Load Phase**
   - Reads JSON manifest file
   - Validates against WinGet schema (v2.0)
   - Extracts package identifiers

2. **Query Phase**
   - For each package, executes `winget show <PackageID>`
   - Retrieves current package name and latest version
   - Populates grid with application details

3. **Installation Phase**
   - Iterates through selected applications
   - Executes: `winget install --id <PackageID> --exact --silent --accept-source-agreements --accept-package-agreements`
   - Tracks success/failure status for each package
   - Generates installation summary

### JSON Structure

The exported JSON follows WinGet's official schema:

```json
{
  "$schema": "https://aka.ms/winget-packages.schema.2.0.json",
  "CreationDate": "2025-11-16T14:30:45.123Z",
  "Sources": [
    {
      "Packages": [
        { "PackageIdentifier": "7zip.7zip" },
        { "PackageIdentifier": "Mozilla.Firefox" },
        { "PackageIdentifier": "Microsoft.VisualStudioCode" }
      ],
      "SourceDetails": {
        "Argument": "https://cdn.winget.microsoft.com/cache",
        "Identifier": "Microsoft.Winget.Source_8wekyb3d8bbwe",
        "Name": "winget",
        "Type": "Microsoft.PreIndexed.Package"
      }
    }
  ]
}
```

### CSV Structure

The CSV provides a comprehensive audit trail:

```csv
"Name","Id","Version","Source","Exportable"
"7-Zip 24.08 (x64)","7zip.7zip","24.08","winget","Yes"
"Mozilla Firefox","Mozilla.Firefox","132.0.2","winget","Yes"
"Spotify","Spotify.Spotify","","msstore","No - Source unavailable"
```

---

## 🔧 Advanced Configuration

### Default File Paths

The application uses UK date formatting (DD-MM-YY):
- Default export location: `C:\temp\MyApps_16-11-25.json`
- Automatic CSV generation: `C:\temp\MyApps_16-11-25.csv`

### Customising File Locations

Modify the default path in the script (line 57):

```powershell
$defaultPath = "C:\temp\$defaultFileName"
```

Change to your preferred directory:

```powershell
$defaultPath = "D:\Backups\WinGet\$defaultFileName"
```

### Silent Installation Flags

The import process uses these WinGet flags:
- `--exact` - Match package ID exactly
- `--silent` - No user interaction required
- `--accept-source-agreements` - Auto-accept source agreements
- `--accept-package-agreements` - Auto-accept package agreements

---

## 📊 Use Cases

### Scenario 1: New Computer Setup

**Problem**: You've purchased a new computer and want to install all your favourite applications.

**Solution**:
1. On your old computer: Export applications to a USB drive
2. On your new computer: Copy the JSON file and run the import
3. Deselect any unwanted applications
4. Let WinGet handle the installation

**Time Saved**: Hours of manual downloads and installations

### Scenario 2: System Refresh

**Problem**: Windows is running slowly and you want to perform a clean install.

**Solution**:
1. Export your current applications
2. Perform clean Windows installation
3. Install WinGet on fresh system
4. Import your application list
5. Restore your workflow in minutes

### Scenario 3: IT Department Deployment

**Problem**: Need to deploy standardised applications across 50+ computers.

**Solution**:
1. Configure one "golden" computer with required applications
2. Export the application list
3. Distribute the JSON file to all computers
4. Script the import process for automated deployment

**Benefit**: Consistent environment across all machines

### Scenario 4: Developer Environment Setup

**Problem**: New developer joining team needs 30+ development tools installed.

**Solution**:
1. Maintain a team JSON manifest file in version control
2. New developer clones repository
3. Runs import script
4. Full development environment ready in under an hour

---

## ⚠️ Known Limitations

### Source Compatibility

- **WinGet Source Only**: Only applications originally installed from WinGet can be reinstalled
- **Microsoft Store Apps**: Cannot be exported/imported (use `winget` versions where available)
- **Custom Installers**: Third-party or manual installations won't appear in WinGet

### Application Versions

- **Latest Version**: Import always installs the latest available version, not the original version
- **Version Lock**: To lock versions, manually edit the JSON to include version specifications

### Installation Failures

Common reasons for installation failures:
- Package no longer available in WinGet repository
- Network connectivity issues
- Insufficient permissions (run as Administrator if needed)
- Package conflicts with existing software

---

## 🐛 Troubleshooting

### "winget list command failed"

**Cause**: WinGet is not installed or not in PATH

**Solution**:
```powershell
# Check if WinGet is installed
winget --version

# If not installed, install via Microsoft Store:
# Search for "App Installer"
```

### "Schema validation failed"

**Cause**: JSON file structure doesn't match WinGet's expected format

**Solution**: Re-export using the latest version of this script

### Progress bar stuck during import

**Cause**: Individual package installation taking longer than expected

**Solution**: Check the verbose output window for current status. Some packages (like Visual Studio) can take 10+ minutes.

### "Access Denied" errors

**Cause**: Insufficient permissions for installation

**Solution**: Right-click PowerShell and select "Run as Administrator"

---

## 🤝 Contributing

Contributions are welcome! Areas for improvement:

- [ ] Add version pinning support
- [ ] Implement multi-threaded installations
- [ ] Add rollback functionality
- [ ] Create scheduled export automation
- [ ] Support for multiple source types
- [ ] Application dependency detection

### Development Setup

1. Fork the repository
2. Make your changes
3. Test with `PowerShell ISE` or `Visual Studio Code`
4. Submit a pull request

---

## 📝 Changelog

### Version 1.0.0 (2025-11-16)
- ✨ Initial release
- ✨ Export functionality with JSON and CSV output
- ✨ Interactive grid view for selective installation
- ✨ Real-time progress tracking
- ✨ UK date formatting
- ✨ Batch select/deselect operations

---

## 📄 License

This project is licensed under the MIT License - see below for details:

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🔗 Useful Links

- [WinGet Official Documentation](https://learn.microsoft.com/en-us/windows/package-manager/winget/)
- [WinGet Package Repository](https://github.com/microsoft/winget-pkgs)
- [WinGet Community Discord](https://discord.gg/winget)
- [Report Issues](https://github.com/yourusername/winget-app-manager/issues)

---

## 💬 Support

If you find this tool helpful, please:
- ⭐ Star this repository
- 🐛 Report bugs via GitHub Issues
- 💡 Suggest features via GitHub Discussions
- 📢 Share with colleagues who might benefit

---

**Made with ❤️ for the Windows community**
