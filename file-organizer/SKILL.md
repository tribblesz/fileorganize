---
name: file-organizer

description: Intelligently organizes files and folders by analyzing content, finding duplicates, suggesting structures, renaming files based on content, and maintaining a markdown index of the directory.
---

#instructions:
File Organizer
This skill acts as your personal organization assistant, helping you maintain a clean, logical file structure across your computer (compatible with Windows, macOS, and Linux). It reduces the cognitive load of manual organization by analyzing file contents, identifying duplicates, proposing directory structures, intelligently renaming files based on their actual content, and generating a tracking index to keep everything searchable.
When to Use This Skill
•	Your Downloads folder is a chaotic mess
•	Files lack descriptive names and require renaming based on their internal content
•	You need a searchable index of what files exist and what they contain
•	You have duplicate files taking up space
•	Your folder structure doesn't make sense anymore
•	You want to establish better organization habits
•	You're starting a new project and need a good structure
•	You're cleaning up before archiving old projects
Important Note for Claude Code Users
If you are running this skill via Claude Code and experience PostToolUse:Bash hook error messages, this is a known issue in Claude Code (Issue #33656) where hooks misreport errors if the underlying Bash command exits non-zero (e.g., when a find command encounters a permission denied error on a single file).
To prevent this, ensure your commands suppress non-critical errors or always exit 0 if the error is expected. The instructions below have been designed to minimize these false positives.
What This Skill Does
1.	Analyzes Current Structure: Reviews your folders and files to understand what you have
2.	Inspects File Content: Reads the actual contents of documents to understand their purpose
3.	Intelligent Renaming: Suggests and applies new, highly descriptive filenames based on the extracted content
4.	Finds Duplicates: Identifies duplicate files across your system safely using hashes
5.	Suggests Organization: Proposes logical folder structures based on your content
6.	Automates Cleanup: Moves, renames, and organizes files with your approval
7.	Maintains Content Index: Generates and updates an index.md file that tracks the contents, old names, new names, and summaries of the organized directory
How to Use
From Your Home Directory
On Windows (PowerShell):
powershell
cd $HOME
On macOS/Linux:
bash
cd ~
Then ask for help:

Help me organize my Downloads folder, rename the files based on what's inside them, and create an index.

Find duplicate files in my Documents folder and help me clean it up.

Review my project directories, suggest improvements, and build a content index for me.
Instructions
When a user requests file organization help:
1. Understand the Scope
Ask clarifying questions:
•	Which directory needs organization? (Downloads, Documents, entire home folder?)
•	What's the main problem? (Can't find things, duplicates, too messy, need better names?)
•	Any files or folders to avoid? (Current projects, sensitive data?)
•	How aggressively to organize? (Conservative vs. comprehensive cleanup)
2. Analyze Current State
Review the target directory using the appropriate commands for the user's OS:
On Windows (PowerShell):
powershell
# Get overview of current structure
Get-ChildItem -Path [target_directory] -ErrorAction Ignore

# Identify largest files
Get-ChildItem -Path [target_directory] -File -ErrorAction Ignore |
  Sort-Object Length -Descending |
  Select-Object -First 20 Name,
    @{Name='Size(MB)'; Expression={ [math]::Round($_.Length / 1MB, 2) }},
    LastWriteTime |
  Format-Table -AutoSize

# Count file types
Get-ChildItem -Path [target_directory] -File -ErrorAction Ignore |
  Group-Object -Property Extension |
  Sort-Object Count -Descending |
  Select-Object Count, Name
On macOS/Linux (Bash):
bash
# Get overview of current structure
ls -la [target_directory]

# Check file types and sizes (suppress permission errors to avoid Claude Code hook failures)
find [target_directory] -type f -exec file {} + 2>/dev/null | head -20

# Identify largest files safely
find [target_directory] -maxdepth 1 -type f -exec du -sh {} + 2>/dev/null | sort -rh | head -20

# Count file types
find [target_directory] -type f 2>/dev/null | sed 's/.*\.//' | sort | uniq -c | sort -rn
Summarize findings:
•	Total files and folders
•	File type breakdown
•	Size distribution
•	Date ranges
•	Obvious organization issues
3. Inspect Content and Propose Renaming
For files with generic or non-descriptive names (e.g., document1.pdf, untitled.txt, IMG_1234.jpg), inspect their contents to determine a suitable name.
•	Use appropriate tools to read the file content (e.g., Get-Content or cat for text, pdftotext for PDFs, or vision models for images).
•	Determine a descriptive name using a consistent format, such as YYYY-MM-DD - Topic - Description.ext.
•	Crucial: Keep track of the original filename so it can be logged in the index later.
4. Find Duplicates
When requested, search for duplicates safely using cryptographic hashes:
On Windows (PowerShell):
powershell
# Find duplicates by hash (ErrorAction Ignore prevents Claude Code hook errors on locked files)
Get-ChildItem -Path [directory] -File -Recurse -ErrorAction Ignore |
  Get-FileHash -ErrorAction Ignore |
  Group-Object -Property Hash |
  Where-Object { $_.Count -gt 1 } |
  ForEach-Object { $_.Group | Select-Object Path, Hash }
On macOS/Linux (Bash):
bash
# Find exact duplicates by hash (use md5sum on Linux, md5 on macOS)
# Suppress stderr to prevent Claude Code PostToolUse hook errors on locked files
find [directory] -type f -exec md5sum {} + 2>/dev/null | sort | uniq -w32 -dD
For each set of duplicates:
•	Show all file paths
•	Display sizes and modification dates
•	Recommend which to keep (usually newest or best-named)
•	Important: Always ask for confirmation before deleting
5. Propose Organization Plan
Present a clear plan before making changes:
markdown
# Organization Plan for [Directory]

## Current State
- X files across Y folders
- [Size] total
- File types: [breakdown]
- Issues: [list problems]

## Proposed Structure

[Directory]/
├── Work/
│   ├── Projects/
│   ├── Documents/
│   └── Archive/
├── Personal/
│   ├── Photos/
│   ├── Documents/
│   └── Media/
└── Downloads/
    ├── To-Sort/
    └── Archive/

## Changes I'll Make

1. **Create new folders**: [list]
2. **Rename files based on content**:
   - `untitled_doc.pdf` → `2024-10-20 - Q4 Financial Report.pdf`
   - `scan001.jpg` → `2024-09-15 - Tax Return Receipt.jpg`
3. **Move files**:
   - X PDFs → Work/Documents/
   - Y images → Personal/Photos/
   - Z old files → Archive/
4. **Delete**: [duplicates or trash files]
5. **Create Index**: Generate `index.md` tracking all contents and changes.

## Files Needing Your Decision

- [List any files you're unsure about]

Ready to proceed? (yes/no/modify)
6. Execute Organization
After approval, organize systematically using the appropriate OS commands:
On Windows (PowerShell):
powershell
# Create folder structure
New-Item -ItemType Directory -Force -Path "path\to\new\folders"

# Move and rename files
Move-Item -Path "old\path\untitled_doc.pdf" -Destination "new\path\2024-10-20 - Q4 Financial Report.pdf"
On macOS/Linux (Bash):
bash
# Create folder structure
mkdir -p "path/to/new/folders"

# Move and rename files with clear logging
mv "old/path/untitled_doc.pdf" "new/path/2024-10-20 - Q4 Financial Report.pdf"
Important Rules:
•	Always confirm before deleting anything
•	Log all moves in memory to build the index
•	Preserve original modification dates where possible
•	Handle filename conflicts gracefully (e.g., append -1)
•	Stop and ask if you encounter unexpected situations or permissions errors
•	CRITICAL: When using Bash to move or rename files, use variables for paths to avoid shell quoting errors with apostrophes (e.g., src="file.pdf"; dest="Anomalou's.pdf"; mv "$src" "$dest"). Never use double quotes directly around paths with apostrophes inside a bash -c call, as it will cause an unexpected EOF error.
7. Generate Content Index (index.md)
After the files are organized and renamed, generate a markdown file named index.md in the root of the organized directory. This file serves as a persistent tracking document.
The index.md must contain:
•	The date of the last organization run.
•	A visual representation of the directory tree.
•	A catalog of files, including their new names, original names, and a brief 1-2 sentence summary of their content extracted during step 3.
Example index.md structure:
markdown
# Directory Index

**Last Updated:** 2024-10-25

## Structure
- /Work
  - 2024-10-20 - Q4 Financial Report.pdf
- /Personal
  - 2024-09-15 - Tax Return Receipt.jpg

## File Catalog

### Work
- **2024-10-20 - Q4 Financial Report.pdf** (formerly `untitled_doc.pdf`)
  - *Summary*: The final Q4 financial report detailing revenue, expenses, and projected growth for the upcoming fiscal year.

### Personal
- **2024-09-15 - Tax Return Receipt.jpg** (formerly `scan001.jpg`)
  - *Summary*: Scanned receipt for tax preparation services from H&R Block.
8. Provide Summary and Maintenance Tips
After organizing and indexing:
markdown
# Organization Complete! ✨

## What Changed

- Created [X] new folders
- Renamed [Y] files based on their content
- Organized [Z] files into logical directories
- Freed [W] GB by removing duplicates
- Generated `index.md` to track all files and their summaries

## New Structure

[Show the new folder tree]

## Maintenance Tips

To keep this organized:

1. **Weekly**: Sort new downloads and run the organizer to update `index.md`
2. **Monthly**: Review and archive completed projects
3. **Quarterly**: Check for new duplicates
4. **Yearly**: Archive files older than 15 years and clean up the structure

Want to organize another folder?
