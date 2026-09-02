### This a is a auto delete script

## Overview

This PowerShell script automatically deletes specific image files (`.jpg` and `.png`) that are older than a defined retention period (`10 days`) from a designated folder and logs all actions and errors to a text file.

---

## Configuration Variables

* `$TargetFolder`: Specifies the directory path being monitored (`E:\test\old data`).
* `$DaysToKeep`: Defines the age threshold in days (`10`) for file retention.
* `$LogFile`: Defines the file path where execution logs are recorded (`E:\test\deletion_log.txt`).

---

## Execution Workflow

1. **Calculate Cutoff Date**: Determines the exact threshold timestamp (`10 days ago` from midnight) using `(Get-Date).AddDays(-$DaysToKeep).Date`.
2. **Initialize Log**: Appends a header block to the log file indicating the task start time, target folder, and cutoff date.
3. **Directory Verification**: Checks if `$TargetFolder` exists using `Test-Path`.
* **If the folder does not exist**: Logs a critical error message and terminates the task.
* **If the folder exists**: Proceeds to scan for files.


4. **File Selection**: Recursively fetches files (`-Recurse -File`) matching the extensions `*.jpg` and `*.png` (`-Include`) whose last write time is older than the cutoff date (`$_.LastWriteTime -lt $CutoffDate`).
5. **Processing Files**:
      **If no matching files are found**: Logs that no files older than the specified days were found.
      **If matching files are found**: Iterates through each file, attempts to delete it forcefully (`Remove-Item -Path $File.FullName -Force -ErrorAction Stop`), and logs success with the file's path and last modified timestamp. If an error occurs during deletion, the exception is caught and logged as an error message.

6. **Task Completion**: Appends a footer block with the completion timestamp to close out the log entry.

```bash
# --- CONFIGURATION ---
$TargetFolder = "E:\test\old data"
$DaysToKeep   = 10
$LogFile      = "E:\test\deletion_log.txt"
# ---------------------

# Calculate the cutoff date (exactly 10 days ago from midnight)
$CutoffDate = (Get-Date).AddDays(-$DaysToKeep).Date

# Start Logging
Add-Content -Path $LogFile -Value "=========================================="
Add-Content -Path $LogFile -Value "Deletion Task Started: $(Get-Date)"
Add-Content -Path $LogFile -Value "Target Folder: $TargetFolder"
Add-Content -Path $LogFile -Value "Deleting files older than: $CutoffDate"
Add-Content -Path $LogFile -Value "=========================================="

# Check if the directory exists
if (Test-Path -Path $TargetFolder) {
    
    # Fetch files older than the cutoff date (excluding folders themselves)
    $OldFiles = Get-ChildItem -Path $TargetFolder -Recurse -File -Include *.jpg,*.png | Where-Object { $_.LastWriteTime -lt $CutoffDate }

    if ($OldFiles.Count -eq 0) {
        Add-Content -Path $LogFile -Value "No files found older than $DaysToKeep days."
    } else {
        foreach ($File in $OldFiles) {
            try {
                # Delete the file
                Remove-Item -Path $File.FullName -Force -ErrorAction Stop
                Add-Content -Path $LogFile -Value "DELETED: $($File.FullName) [Modified: $($File.LastWriteTime)]"
            }
            catch {
                Add-Content -Path $LogFile -Value "ERROR: Failed to delete $($File.FullName). Reason: $_"
            }
        }
    }
} else {
    Add-Content -Path $LogFile -Value "CRITICAL ERROR: Target folder '$TargetFolder' does not exist."
}

# End Logging
Add-Content -Path $LogFile -Value "Deletion Task Finished: $(Get-Date)`n"
```
