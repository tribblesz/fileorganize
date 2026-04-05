name: file-organizer
description: Intelligently organizes files and folders by analyzing content, finding duplicates, suggesting structures, renaming files based on content, and maintaining a persistent memory index to manage context length.
#instructions:
File Organizer
This skill acts as your personal organization assistant, helping you maintain a clean, logical file structure across your computer. It reduces cognitive load by analyzing file contents in chunks (to prevent context window overflow), proposing directory structures, intelligently renaming files, and generating a persistent memory.md tracker to keep knowledge accessible without exceeding context limits.
Important Note for Claude Code Users
If you experience PostToolUse:Bash hook error messages, this is a known issue (Issue #33656) where hooks misreport errors if the underlying Bash command exits non-zero (e.g., when a find command encounters a permission denied error). Ensure your commands suppress non-critical errors or always exit 0 if the error is expected.
What This Skill Does
1.	Analyzes Structure: Reviews folders and files to understand current state.
2.	Context-Aware Content Inspection: Reads file contents in small, controlled chunks to prevent context window truncation.
3.	Persistent Memory Tracking: Writes file summaries and metadata to a persistent memory.md file, allowing you to drop file contents from active context while retaining the knowledge.
4.	Intelligent Renaming: Applies new, highly descriptive filenames based on the extracted content, updating the memory.md tracker to link old and new names.
5.	Finds Duplicates: Identifies duplicates safely using hashes.
6.	Automates Cleanup: Moves, renames, and organizes files.
7.	Maintains Content Index: Generates an index.md for user visibility, while relying on memory.md for its own internal state tracking.
Instructions
When a user requests file organization help, you MUST follow these steps in order:
1. Establish Operation Mode (Crucial First Step)
Before taking any actions, you MUST ask the user which operation mode they prefer:
**"Before we begin organizing, please choose an operation mode:
1.	Stop-Point Mode: I will pause and ask for your explicit approval before making any changes (moving, renaming, or deleting files).
2.	Everything about Stop-Point Mode, but I should have the option at each stage to switch fully to Autonomous Mode.
3.	Autonomous Mode: I will make all decisions autonomously and execute the entire organization process without interrupting you."**
Record the user's choice and strictly adhere to it.
2. Initialize Persistent Memory Tracker (memory.md)
To manage context length effectively, you must establish a persistent memory file in the root of the target directory.
•	Create or read the file named memory.md.
•	This file acts as your external brain. Once you summarize a file and write it to memory.md, you no longer need to keep the raw file contents in your active context.
Format for memory.md:
markdown
# File Organizer Memory Tracker

| Original Name | Current Name | Hash/ID | Summary of Content | Status |
|---|---|---|---|---|
| `untitled.pdf` | `2024-10-20 - Q4 Report.pdf` | `a1b2c3d4` | Q4 financial report detailing revenue... | Organized |
3. Context-Aware Content Inspection (Chunking)
When reading files to determine what they are, DO NOT read the entire file at once if it is large, as this will blow out your context window and cause truncation.
•	Chunking Strategy: Use commands like head -n 50 [file] or PowerShell's Get-Content -TotalCount 50 to read only the first chunk of a file.
•	If the first chunk does not contain enough information to generate a descriptive name, read the next chunk (e.g., sed -n '51,100p' [file]).
•	Once you have enough information to summarize the file, STOP reading.
•	Immediately write the 1-2 sentence summary to memory.md and drop the raw file text from your active reasoning context.
4. Find Duplicates
Search for duplicates safely using cryptographic hashes:
On Windows (PowerShell):
powershell
Get-ChildItem -Path [directory] -File -Recurse -ErrorAction Ignore | Get-FileHash -ErrorAction Ignore | Group-Object -Property Hash | Where-Object { $_.Count -gt 1 } | ForEach-Object { $_.Group | Select-Object Path, Hash }
On macOS/Linux (Bash):
bash
find [directory] -type f -exec md5sum {} + 2>/dev/null | sort | uniq -w32 -dD
5. Propose Organization Plan
Using the knowledge stored in memory.md, present a clear plan. If in Stop-Point Mode, wait for approval. If in Autonomous Mode, proceed immediately.
6. Execute Organization and Update Trackers
Execute systematically using the appropriate OS commands:
On Windows (PowerShell):
powershell
New-Item -ItemType Directory -Force -Path "path\to\new\folders"
Move-Item -Path "old\path\untitled_doc.pdf" -Destination "new\path\2024-10-20 - Q4 Financial Report.pdf"
On macOS/Linux (Bash):
bash
mkdir -p "path/to/new/folders"
mv "old/path/untitled_doc.pdf" "new/path/2024-10-20 - Q4 Financial Report.pdf"
CRITICAL Execution Rules:
•	Apostrophes: When using Bash to move or rename files, use variables for paths to avoid shell quoting errors with apostrophes (e.g., src="file.pdf"; dest="Anomalou's.pdf"; mv "$src" "$dest"). Never use double quotes directly around paths with apostrophes inside a bash -c call.
•	Tracker Updates: As soon as a file is renamed or moved, you MUST update the Current Name and Status columns in memory.md so you do not lose track of the file.
7. Generate User-Facing Content Index (index.md)
After organization is complete, generate a clean, user-friendly index.md from the data in memory.md.
Self-Monitoring and Loop Detection
As an AI agent executing this skill, you MUST actively monitor your own behavior:
1.	Loop Detection: If you notice you have executed the same command or received the same error more than 3 times in a row, YOU ARE IN A LOOP.
2.	Self-Repair: Upon detecting a loop, immediately STOP executing the failing command.
3.	Skill Modification: If the loop is caused by a fundamental flaw in how this skill is written or interpreted, you must edit this skill file in real-time to fix the issue.
4.	Skill Placement: Ensure this skill file (FileOrganizer.md or file-organizer.md) is always saved and maintained at the top level of the working folder, directly next to the CLAUDE.md file, so it remains easily accessible and up-to-date for future sessions.
