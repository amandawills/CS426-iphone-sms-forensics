# CS426-iphone-sms-forensics

# Recovery and Analysis of iPhone SMS Messages from a Public Forensic Image

This project demonstrates how to recover and analyze iPhone SMS/iMessage artifacts from a public forensic image using a macOS-friendly workflow. Instead of relying on Autopsy or FTK Imager, this guide uses **DB Browser for SQLite** to directly inspect the iPhone message database (`sms.db`).

## Project Overview

The goal of this project is to:

- acquire a public iPhone forensic dataset
- locate the iPhone SMS database
- open and inspect `sms.db`
- extract messages, timestamps, and contacts
- reconstruct a basic communication timeline

This workflow is designed to be simple, reproducible, and suitable for a digital forensics class project.

## Dataset Used

This project uses the **2020 iOS Magnet CTF** dataset from the **NIST CFReDS Mobile Device Images** collection.

Source:
- NIST CFReDS Mobile Device Images

Recommended dataset:
- `2020 iOS – Magnet CTF`

After extraction, the iPhone file system should contain folders such as:

```text
Applications
Library
private
System
usr
```

Environment
Hardware
MacBook or other desktop/laptop
At least 8 GB RAM recommended
At least 10–20 GB free storage
Software
macOS
Homebrew
DB Browser for SQLite
Optional: Python 3
Step 1: Install DB Browser for SQLite
Install with Homebrew:
brew install --cask db-browser-for-sqlite
Open the application:
open -a "DB Browser for SQLite"
Step 2: Download the Dataset
Go to the NIST CFReDS Mobile Device Images page.
Download the 2020 iOS – Magnet CTF dataset.
Save it locally.
Extract the archive.
After extraction, place the dataset in a convenient location, such as your Desktop.
Example:
~/Desktop/iOS_Filesystem
Step 3: Locate the SMS Database
Inside the extracted iPhone file system, navigate to:
/private/var/mobile/Library/SMS/
You should find files similar to:
sms.db
sms.db-shm
sms.db-wal
The main forensic artifact used in this project is:
sms.db
Step 4: Open the Database
Launch DB Browser for SQLite
Click Open Database
Select the file:
/private/var/mobile/Library/SMS/sms.db
Open the Browse Data tab
Relevant tables commonly include:
message
handle
chat
chat_message_join
Step 5: Verify the Database Contents
In Browse Data, choose the message table.
Important fields may include:
text
date
handle_id
is_from_me
These fields help identify:
message content
when the message was sent or received
which contact is associated with the message
whether the device user sent the message
Step 6: Run Core SQL Queries
Query 1: Basic Timeline Reconstruction
This query converts Apple's timestamp format into a readable datetime and links messages to contacts.
SELECT
    datetime(message.date/1000000000 + 978307200,'unixepoch') AS date,
    message.text,
    handle.id AS contact,
    message.is_from_me
FROM message
LEFT JOIN handle
ON message.handle_id = handle.rowid
ORDER BY message.date;
Query 2: Sent vs. Received Message Count
SELECT
    CASE
        WHEN is_from_me = 1 THEN 'Sent'
        ELSE 'Received'
    END AS direction,
    COUNT(*) AS total_messages
FROM message
GROUP BY is_from_me;
Query 3: Most Frequent Contacts
SELECT
    handle.id AS contact,
    COUNT(*) AS message_count
FROM message
LEFT JOIN handle
ON message.handle_id = handle.rowid
GROUP BY handle.id
ORDER BY message_count DESC;
Query 4: Sample Non-Empty Messages
SELECT
    datetime(date/1000000000 + 978307200,'unixepoch') AS date,
    text
FROM message
WHERE text IS NOT NULL
  AND text != ''
ORDER BY date
LIMIT 10;
Step 7: Interpret Key Fields
is_from_me
1 = sent from the device
0 = received by the device
handle.id
This typically represents the phone number or Apple ID associated with the conversation.
date
Apple stores timestamps in a different format. The SQL conversion used above turns the raw value into a readable timestamp.
Step 8: Export Results
To save query results:
Run a query in the Execute SQL tab
Export the results as CSV
Save the file for reporting or visualization
Suggested exports:
message timeline
sent vs. received counts
most frequent contacts
Step 9: Document Evidence for Class Deliverables
Capture screenshots of:
the extracted dataset folder
the /private/var/mobile/Library/SMS/ directory
the sms.db file
DB Browser showing the message table
SQL query results
These screenshots can be used in:
setup readiness reports
milestone demos
final presentations
final reports
Step 10: Present the Project
Suggested Live Demo Flow
Show the extracted iPhone file system
Navigate to /private/var/mobile/Library/SMS/
Open sms.db in DB Browser for SQLite
Show the message table
Run the timeline query
Run the frequent contacts query
Explain how the results support forensic analysis
Findings You Can Discuss
Using this workflow, you can demonstrate:
successful acquisition of a public forensic dataset
identification of the iPhone SMS database
recovery of message content and metadata
timeline reconstruction from message timestamps
contact-based communication analysis
Limitations
Some datasets may not contain deleted message remnants
Attachments and chat relationships may require deeper table joins
Full forensic suites may provide additional artifact parsing
Message interpretation depends on the completeness of the dataset
Optional Enhancements
If expanding the project, you can also:
investigate attachments
map messages to chat threads
export results into Python or Excel
build message timelines or graphs
search for keywords in message content
Repository Structure
Suggested structure for this project:
.
├── README.md
├── queries/
│   └── sms_queries.sql
├── screenshots/
│   ├── dataset_folder.png
│   ├── sms_directory.png
│   ├── sqlite_message_table.png
│   └── query_results.png
└── notes/
    └── project_notes.md
Disclaimer
This project uses a public forensic training dataset intended for educational and research purposes. Do not use private device data without proper authorization.
Author
Created as part of a digital forensics course project on iPhone SMS artifact analysis.
