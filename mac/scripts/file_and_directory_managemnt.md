### File and Directory Management 

##### This script is used to file and directory management.

##### Steps involved in the process!

1. Create the today's work folder.
2. Copy files from project/template files to today's work folder.
3. Rename the report files in the created folder with a timestamp.
4. If a work folder exsists, compress it and copy to the backup directory.
5. Sync project folder to backup.
6. Generate a directory report.

```
# Define directories and variables
PROJECT_DIR="$HOME/Projects"
BACKUP_DIR="$HOME/ProjectBackups"
TEMPLATE_DIR="$PROJECT_DIR/templates"
TODAY=$(date +"%Y-%m-%d")
TODAY_DIR="$PROJECT_DIR/Work_$TODAY"

# Ensure backup directory exists
mkdir -p "$BACKUP_DIR/latest"

# Step 1. Create the today's work folder.
echo "Commencing today's work folder creation process!"
mkdir -p "$TODAY_DIR"
echo "Ending today's work folder creation process!"

# Step 2. Copy files from project/template files to today's work folder.
echo "Commencing copying project templates to the todays work folder!"
if [ -d "$TEMPLATE_DIR" ]; then
cp -R "$TEMPLATE_DIR/"* "$TODAY_DIR"/ 2>/dev/null
else
echo "No template directory found at $TEMPLATE_DIR"
fi
echo "Ending copying project templates to the todays work folder!"

# Step 3. Rename the report files in the created folder with a timestamp.
echo "Commencing renaming files in the folder with a timestamp."
for file in "$TODAY_DIR"/report*.txt; do
[[ -e "$file" ]] || continue
timestamp=$(date +"%H%M%S")
mv "$file" "${file%.txt}_$timestamp.txt"
done
echo "Ending renaming files process!"

# Step 4. If a work folder exsists, compress it and copy to the backup directory
echo "Commencing compressing files and copying to the backup directory process!"
YESTERDAY=$(ls -td "$PROJECT_DIR"/Work_* 2>/dev/null | sed -n '2p')
if [[ -n "$YESTERDAY" && -d "$YESTERDAY" ]]; then
zip -r -q "$BACKUP_DIR/$(basename "$YESTERDAY").zip" "$YESTERDAY"
else
echo "No previous Work_ folder found to compress."
fi
echo "Ending compressing files and copying to the backup directory process!"

# Step 5. Sync project folder to backup.
echo "Commencing project folder syncing process!"
rsync -av --delete "$PROJECT_DIR"/ "$BACKUP_DIR/latest/" >/dev/null 2>&1
echo "Ending project folder syncing process!"

# Step 6. Generate a directory report.
echo "Commencing directory report generation process!"
find "$PROJECT_DIR" -type f -exec ls -lh {} + > "$BACKUP_DIR/directory_report_$TODAY.txt"
echo "Ending directory report generation process!"

echo "All tasks completed successfully!"
```

